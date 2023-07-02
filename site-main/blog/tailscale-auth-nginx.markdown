---
title: "Tailscale Authentication for NGINX"
date: 2022-04-27
redirect_to: https://tailscale.com/blog/tailscale-auth-nginx/
---

<xeblog-conv name="Mimi" mood="happy">This page shows how to use nginx-auth, a tool that implements the NGINX HTTP subrequest authentication protocol, to authenticate requests to internal services proxied behind NGINX with Tailscale. It also explains how to install and configure nginx-auth and NGINX, and how to use the response headers to pass authentication information to the applications. The benefits of this approach are similar to setting up a corporate SSO system without the installation and maintenance overhead.</xeblog-conv>
