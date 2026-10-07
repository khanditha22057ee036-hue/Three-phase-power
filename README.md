"""
Three Phase Power Calculator
----------------------------
A menu-driven program for engineering students to calculate power in a
balanced three-phase system.

Key relations (balanced load):
    Apparent power  S = sqrt(3) * VL * IL          (VA)
    Active power    P = sqrt(3) * VL * IL * cos(phi)   (W)
    Reactive power  Q = sqrt(3) * VL * IL * sin(phi)   (VAR)
    Power factor    pf = cos(phi) = P / S

Star (Y) connection:    VL = sqrt(3) * Vph,   IL = Iph
Delta connection:       VL = Vph,             IL = sqrt(3) * Iph

Two-wattmeter method:   P = W1 + W2
                        tan(phi) = sqrt(3) * (W1 - W2) / (W1 + W2)

Power factor correction:
                        Qc = P * (tan(phi1) - tan(phi2))
"""

import math

SQRT3 = math.sqrt(3)


# ---------------------------------------------------------------- input helpers
def get_positive_float(prompt):
    """Keep asking until the user enters a valid positive number."""
    while True:
        try:
            value = float(input(prompt))
            if value <= 0:
                print("  Please enter a value greater than zero.")
                continue
            return value
        except ValueError:
            print("  Invalid input. Please enter a number.")


def get_power_factor(prompt="Power factor (0 to 1): "):
    """Ask for a power factor between 0 (exclusive) and 1 (inclusive)."""
    while True:
        pf = get_positive_float(prompt)
        if pf <= 1:
            return pf
        print("  Power factor cannot be greater than 1.")


# ------------------------------------------------------------------ core maths
def apparent_power(vl, il):
    """S = sqrt(3) * VL * IL  (VA)"""
    return SQRT3 * vl * il


def active_power(vl, il, pf):
    """P = sqrt(3) * VL * IL * cos(phi)  (W)"""
    return SQRT3 * vl * il * pf


def reactive_power(vl, il, pf):
    """Q = sqrt(3) * VL * IL * sin(phi)  (VAR)"""
    return SQRT3 * vl * il * math.sqrt(1 - pf ** 2)


def line_current(p, vl, pf):
    """IL = P / (sqrt(3) * VL * cos(phi))"""
    return p / (SQRT3 * vl * pf)


def star_to_line(v_ph, i_ph):
    """Star connection: VL = sqrt(3) * Vph, IL = Iph"""
    return SQRT3 * v_ph, i_ph


def delta_to_line(v_ph, i_ph):
    """Delta connection: VL = Vph, IL = sqrt(3) * Iph"""
    return v_ph, SQRT3 * i_ph


def two_wattmeter(w1, w2):
    """Return total power and power factor from two wattmeter readings."""
    total = w1 + w2
    if total == 0:
        return 0.0, 0.0
    phi = math.atan(SQRT3 * (w1 - w2) / total)
    return total, math.cos(phi)


def capacitor_kvar(p_kw, pf_old, pf_new):
    """Capacitor rating (kVAR) to improve power factor from pf_old to pf_new."""
    phi1 = math.acos(pf_old)
    phi2 = math.acos(pf_new)
    return p_kw * (math.tan(phi1) - math.tan(phi2))


def efficiency(p_out, p_in):
    """Efficiency (%) = output / input * 100"""
    return p_out / p_in * 100


# ------------------------------------------------------------------- display
def print_power_triangle(vl, il, pf):
    s = apparent_power(vl, il)
    p = active_power(vl, il, pf)
    q = reactive_power(vl, il, pf)
    print("\n  ----- Results -----")
    print(f"  Apparent power S = {s:,.2f} VA  ({s / 1000:.3f} kVA)")
    print(f"  Active power   P = {p:,.2f} W   ({p / 1000:.3f} kW)")
    print(f"  Reactive power Q = {q:,.2f} VAR ({q / 1000:.3f} kVAR)")
    print(f"  Power factor     = {pf:.3f}")
    print(f"  Phase angle      = {math.degrees(math.acos(pf)):.2f} degrees")


def menu():
    print("\n" + "=" * 52)
    print("        THREE PHASE POWER CALCULATOR")
    print("=" * 52)
    print(" 1. Power from line voltage, line current, p.f.")
    print(" 2. Power from phase values (Star connection)")
    print(" 3. Power from phase values (Delta connection)")
    print(" 4. Line current from power, voltage, p.f.")
    print(" 5. Power by two-wattmeter method")
    print(" 6. Capacitor rating for power factor correction")
    print(" 7. Efficiency of a three-phase machine")
    print(" 0. Exit")
    print("-" * 52)


def main():
    while True:
        menu()
        choice = input("Enter your choice: ").strip()

        if choice == "1":
            vl = get_positive_float("Line voltage VL (V): ")
            il = get_positive_float("Line current IL (A): ")
            pf = get_power_factor()
            print_power_triangle(vl, il, pf)

        elif choice == "2":
            v_ph = get_positive_float("Phase voltage Vph (V): ")
            i_ph = get_positive_float("Phase current Iph (A): ")
            pf = get_power_factor()
            vl, il = star_to_line(v_ph, i_ph)
            print(f"\n  Line voltage VL = {vl:.2f} V, Line current IL = {il:.2f} A")
            print_power_triangle(vl, il, pf)

        elif choice == "3":
            v_ph = get_positive_float("Phase voltage Vph (V): ")
            i_ph = get_positive_float("Phase current Iph (A): ")
            pf = get_power_factor()
            vl, il = delta_to_line(v_ph, i_ph)
            print(f"\n  Line voltage VL = {vl:.2f} V, Line current IL = {il:.2f} A")
            print_power_triangle(vl, il, pf)

        elif choice == "4":
            p = get_positive_float("Active power P (W): ")
            vl = get_positive_float("Line voltage VL (V): ")
            pf = get_power_factor()
            print(f"\n  Line current IL = {line_current(p, vl, pf):.3f} A")

        elif choice == "5":
            w1 = get_positive_float("Wattmeter reading W1 (W): ")
            w2 = get_positive_float("Wattmeter reading W2 (W): ")
            total, pf = two_wattmeter(w1, w2)
            print(f"\n  Total power P = {total:,.2f} W")
            print(f"  Power factor  = {pf:.3f}")

        elif choice == "6":
            p_kw = get_positive_float("Load active power (kW): ")
            pf_old = get_power_factor("Existing power factor: ")
            pf_new = get_power_factor("Desired power factor: ")
            if pf_new <= pf_old:
                print("  Desired power factor must be higher than the existing one.")
            else:
                print(f"\n  Capacitor bank required = {capacitor_kvar(p_kw, pf_old, pf_new):.2f} kVAR")

        elif choice == "7":
            p_out = get_positive_float("Output power (W): ")
            p_in = get_positive_float("Input power (W): ")
            if p_out > p_in:
                print("  Output power cannot exceed input power.")
            else:
                print(f"\n  Efficiency = {efficiency(p_out, p_in):.2f} %")

        elif choice == "0":
            print("\nThank you for using the calculator. Goodbye!")
            break

        else:
            print("  Invalid choice. Please select from the menu.")


if __name__ == "__main__":
    main()
