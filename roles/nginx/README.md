# nginx

## How to Run

Typically you'd use a command similar to this to run this role:

    ansible-playbook -i inventory playbooks/nginx.yml -e "dns_cloudflare_api_token=<your_token_here>" -e "letsencrypt_email=<your_email_here>"

## Dependencies

Running this role also activates `certbot` role. 

## Tested OS

This role is tested on following operating systems:

- Ubuntu 22.04
