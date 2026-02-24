#!/usr/bin/env -S uv run --quiet --script
# /// script
# requires-python = ">=3.10"
# ///

import json
import os
import re
import subprocess
import sys
from pathlib import Path

CONFIG_DIR = Path.home() / ".config" / "isim"
DEFAULT_FILE = CONFIG_DIR / "default"

BOLD = "\033[1m"
DIM = "\033[2m"
CYAN = "\033[0;36m"
GREEN = "\033[0;32m"
YELLOW = "\033[1;33m"
RESET = "\033[0m"


def get_simulators() -> list[dict]:
    result = subprocess.run(
        ["xcrun", "simctl", "list", "devices", "available", "--json"],
        capture_output=True, text=True, check=True,
    )
    data = json.loads(result.stdout)
    simulators = []

    for runtime, devices in data["devices"].items():
        m = re.search(r"(i(?:OS|PadOS))-(\d+)-(\d+)(?:-(\d+))?$", runtime, re.IGNORECASE)
        if not m:
            continue
        os_type = m.group(1)
        major, minor = int(m.group(2)), int(m.group(3))
        patch = int(m.group(4)) if m.group(4) else 0
        os_name = f"{os_type} {major}.{minor}" + (f".{patch}" if patch else "")

        for device in devices:
            if device.get("isAvailable", False):
                simulators.append({
                    "udid": device["udid"],
                    "name": device["name"],
                    "os": os_name,
                    "state": device["state"],
                    "version": (major, minor, patch),
                })

    simulators.sort(key=lambda s: (s["version"], s["name"]))
    return simulators


def get_default_udid() -> str | None:
    try:
        return DEFAULT_FILE.read_text().strip()
    except FileNotFoundError:
        return None


def find_simulator(query: str) -> dict | None:
    sims = get_simulators()
    # Exact UDID prefix match first
    for s in sims:
        if s["udid"].lower().startswith(query.lower()):
            return s
    # Substring match on any field
    for s in sims:
        haystack = f"{s['udid']} {s['name']} {s['os']}".lower()
        if query.lower() in haystack:
            return s
    return None


def cmd_list(filter_term: str | None = None):
    sims = get_simulators()

    if filter_term:
        sims = [
            s for s in sims
            if filter_term.lower() in f"{s['udid']} {s['name']} {s['os']} {s['state']}".lower()
        ]

    if not sims:
        msg = "No simulators found"
        if filter_term:
            msg += f" matching '{filter_term}'"
        print(f"{msg}.")
        sys.exit(1)

    default_udid = get_default_udid()
    max_name = max(len(s["name"]) for s in sims)
    max_name = max(max_name, 6)  # at least as wide as "DEVICE"

    header = f"  {'UDID':<36}  {'DEVICE':<{max_name}}  {'OS':<12}  {'STATE':<8}"
    sep = f"  {'─' * 36}  {'─' * max_name}  {'─' * 12}  {'─' * 8}"
    print(f"{BOLD}{CYAN}{header}{RESET}")
    print(f"{DIM}{sep}{RESET}")

    for s in sims:
        prefix = f"{YELLOW}★ {RESET}" if s["udid"] == default_udid else "  "
        color = GREEN if s["state"] == "Booted" else ""
        reset = RESET if color else ""
        line = f"{s['udid']:<36}  {s['name']:<{max_name}}  {s['os']:<12}  {s['state']:<8}"
        print(f"{prefix}{color}{line}{reset}")

    print()
    print(f"  {YELLOW}★{RESET} = default   {GREEN}green{RESET} = booted")


def cmd_launch(query: str):
    sim = find_simulator(query)
    if not sim:
        print(f"No simulator found matching '{query}'.")
        print("Run 'isim list' to see available simulators.")
        sys.exit(1)

    print(f"Launching {BOLD}{sim['name']}{RESET} ({CYAN}{sim['os']}{RESET})...")
    subprocess.run(["open", "-a", "Simulator", "--args", "-CurrentDeviceUDID", sim["udid"]])


def cmd_default_show():
    udid = get_default_udid()
    if not udid:
        print("No default set. Run: isim default <udid>")
        sys.exit(1)

    sim = find_simulator(udid)
    if sim:
        print(f"Default: {BOLD}{sim['name']}{RESET} ({CYAN}{sim['os']}{RESET})")
        print(f"  UDID: {udid}")
    else:
        print(f"Default UDID: {udid} (not found in available simulators)")


def cmd_default_set(udid: str):
    sim = find_simulator(udid)
    if not sim:
        print(f"{YELLOW}Warning:{RESET} UDID '{udid}' not found in available simulators.")
        confirm = input("Set anyway? [y/N] ").strip()
        if confirm.lower() != "y":
            return

    CONFIG_DIR.mkdir(parents=True, exist_ok=True)
    DEFAULT_FILE.write_text(udid + "\n")

    if sim:
        print(f"Default set to: {BOLD}{sim['name']}{RESET} ({CYAN}{sim['os']}{RESET})")
    else:
        print(f"Default set to: {udid}")


def cmd_launch_default():
    udid = get_default_udid()
    if not udid:
        print("No default simulator set.")
        print("Run 'isim list' to browse simulators, then: isim default <udid>")
        sys.exit(1)
    cmd_launch(udid)


def usage():
    print(f"{BOLD}isim{RESET} — iOS Simulator launcher")
    print()
    print(f"{BOLD}Usage:{RESET}")
    print("  isim                      Launch the default simulator")
    print("  isim list [filter]        List available simulators")
    print("  isim launch <query>       Launch by name, OS version, or UDID")
    print("  isim default              Show current default")
    print("  isim default <udid>       Set default simulator")
    print("  isim help                 Show this help")
    print()
    print(f"{BOLD}Examples:{RESET}")
    print("  isim list")
    print("  isim list iphone")
    print("  isim list 'iOS 17'")
    print("  isim launch 'iPhone 15 Pro'")
    print("  isim launch 'iPad 18'")
    print("  isim default A1B2C3D4-E5F6-...")


def main():
    args = sys.argv[1:]
    cmd = args[0] if args else ""

    match cmd:
        case "list":
            cmd_list(args[1] if len(args) > 1 else None)
        case "launch":
            if len(args) < 2:
                print("Usage: isim launch <name|os|udid>")
                sys.exit(1)
            cmd_launch(args[1])
        case "default":
            if len(args) > 1:
                cmd_default_set(args[1])
            else:
                cmd_default_show()
        case "help" | "--help" | "-h":
            usage()
        case "":
            cmd_launch_default()
        case _:
            cmd_launch(cmd)


if __name__ == "__main__":
    main()
