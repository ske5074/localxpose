# Setup rc startup script for Alpine Linux

Alpine Linux is a great lightweight Linux distribution to run web servers on. Use the steps below to create an `rc` startup script to start your tunnels automatically on boot:

## Steps (Run as root)

1. **Install loclx on the system and copy it to `/usr/local/sbin`**
    ```sh
    cp loclx /usr/local/sbin
    chmod 755 /usr/local/sbin/loclx
    ```

2. **Create an `/etc/localxpose` directory and place the `config.yaml` file there**
    ```sh
    mkdir /etc/localxpose
    chmod 750 /etc/localxpose
    ```

3. **Create a `config.yaml` file**  
   Example sections are available [here](https://localxpose.io/docs/cli/config.yaml).
    ```sh
    cat > /etc/localxpose/config.yaml << EOF
    webexample:
      type: http
      region: us
      reserved_domain: www.example.com
      to: localhost:8080
      plugins:
        https_redirect: true
        request_header:
          - host:www.example.com
          - X-Token:secureToken
    minecraft-bedrock:
      type: udp
      region: us
      reserved_endpoint: us.loclx.io:101010
      to: 10.0.1.9:19132
    EOF
    ```

4. **Create the `rc` script**
    ```sh
    cat > /etc/init.d/localxpose << EOF
    #!/sbin/openrc-run
    
    name="localXpose Tunneling Service"
    description="Runs localXpose loclx and restarts it if it stops."
    command="/usr/local/sbin/loclx tunnel config -f /etc/localxpose/config.yaml"
    command_background="yes"
    pidfile="/var/run/$RC_SVCNAME.pid"
    retry="30"
    
    start_pre() {
        checkpath --directory --owner root:root --mode 0755 "$(dirname $pidfile)"
    }
    
    depend() {
        need localmount net
        after firewall
    }
    
    start() {
        ebegin "Starting $name"
        start-stop-daemon --start --background --make-pidfile --pidfile $pidfile --exec $command --retry ${retry}/1/TERM/30/KILL/5
        eend $?
    }
    
    stop() {
        ebegin "Stopping $name"
        start-stop-daemon --stop --pidfile $pidfile
        eend $?
    }
    EOF
    ```

5. **Make the script executable**
    ```sh
    chmod 755 /etc/init.d/localxpose
    ```

6. **Add the service to the default runlevel and start it**
    ```sh
    rc-update add localxpose default
    rc-service localxpose start
    ```
