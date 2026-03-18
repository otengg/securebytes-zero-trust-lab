# Setup Guide

## Install Tailscale
curl -fsSL https://tailscale.com/install.sh | sh

## Authenticate Node
sudo tailscale up --ssh --advertise-tags=tag:monitor --accept-routes

## Subnet Router (Pi-hole)
sudo tailscale up --advertise-routes=192.168.10.0/24 --accept-dns=false

## Enable IP Forwarding
sudo sysctl -w net.ipv4.ip_forward=1

## NAT
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
