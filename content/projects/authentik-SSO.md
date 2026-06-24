+++
date = '2026-06-24T06:02:43Z'
draft = true
title = 'Authentik SSO'
+++

Before implementing Authentik, I was running Adguard Home, the Arr stack (5-6 apps), Jellyfin. Each with its own local account, all sharing one simple password to keep access frictionless on a network I treated as trusted. Absolutely horrible security across the board, honestly. The only meaningful control I had in place was that it's all on my internal network.

Changing the password meant updating it across every service manually. If my network was ever compromised, it wouldn't take long for an attacker to gain full access across the board.

Two key components that I used to solve this problem are Authentik and Caddy. Authentik, an identity provider, manages users, passwords, and their permissions. Authentik is the guard at the gate of my keep, nobody gets in without their okay. Caddy is my reverse proxy, the gate; every request hits it first and then is routed to the right service.

When a request comes in, Caddy checks with Authentik whether the request is authenticated. If it isn't, the request is redirected to the Authentik login page. If the user doesn't exist, enters a wrong password, or doesn't have permission to access the service, they're stopped here. Authentik handles both the authentication (proving identity) and the authorisation (whether this identity has permission to access the service). This is forward auth - Caddy defers the authentication to Authentik and then enforces the result.
Once the user is authenticated with Authentik, Caddy routes the request to the requested service. When a user logs in, they have an authenticated session which means they're able to access the services to which they have permission - a single sign-on.

This design also means that any service that either has a weak login or none at all is still fully protected since authentication happens at the proxy. 
![Forward auth request flow: a request passes through Caddy to the authentik check, unauthenticated requests go to login, authenticated requests reach the services](/img/forward-auth.svg)