# personal-ansible-configs

EC2 is my new home. This is how I orchestrate my fleet.

## Packages

1. **nginx**: Nginx listens on 80. Current configuration expects Jenkins to be running on 8080.
2. **jenkins**: To configure the pipeline from GitHub.
3. **rabbitmq**: Experimental purpose.
4. **elasticsearch**: To track uptime of nodes. Logs from above daemons.
