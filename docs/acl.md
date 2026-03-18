# Tailscale ACL

## Example

{
  "tagOwners": {
    "tag:admin": ["autogroup:admin"],
    "tag:dns": ["autogroup:member"],
    "tag:monitor": ["autogroup:member"]
  },
  "acls": [
    {
      "action": "accept",
      "src": ["tag:admin"],
      "dst": ["192.168.10.0/24:*"]
    },
    {
      "action": "accept",
      "src": ["tag:dns"],
      "dst": ["192.168.10.10:53"]
    },
    {
      "action": "accept",
      "src": ["tag:monitor"],
      "dst": ["192.168.10.20:3000"]
    }
  ]
}
