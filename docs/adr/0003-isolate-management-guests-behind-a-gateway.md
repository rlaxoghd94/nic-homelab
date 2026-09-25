# Isolate management guests behind a gateway

Keep Proxmox host administration on the existing uplink network and place managed guests on one or more isolated Proxmox SDN networks. A multi-homed gateway guest connects the internal networks to the uplink and advertises them through Tailscale, preserving local Proxmox access if the gateway or SDN becomes unavailable.
