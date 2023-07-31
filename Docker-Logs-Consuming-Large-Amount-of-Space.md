If Docker logs are consuming a large amount of space on your system, you might need to periodically clean them up. Here are a few ways you can manage Docker logs:

1. **Limit the Log File Size**: Docker allows you to limit the size of the log files. You can configure Docker to automatically delete older log entries once the limit is reached. This can be done by setting the `max-size` and `max-file` logging options. You can do this in the `docker run` command like so:

   ```bash
   docker run --log-opt max-size=10m --log-opt max-file=3 my_container
   ```
   This command configures Docker to use up to 3 log files that have a maximum size of 10MB each. When the limit is reached, the oldest file will be deleted.

2. **Configure Docker Daemon**: Alternatively, you can configure these options at the Docker daemon level, so you don't have to set it for each container individually. To do this, edit or create the Docker daemon configuration file at `/etc/docker/daemon.json`:

   ```json
   {
     "log-driver": "json-file",
     "log-opts": {
       "max-size": "10m",
       "max-file": "3"
     }
   }
   ```
   Then, restart the Docker daemon to apply the new configuration. On a Unix-based system, this can be done with the command: `systemctl restart docker`.

3. **Manual Log Cleanup**: You can manually remove Docker logs by truncating the log files. Docker logs are stored at `/var/lib/docker/containers/{container-id}/{container-id}-json.log`. You can clean them with the following command:

   ```bash
   echo "" > /var/lib/docker/containers/{container-id}/{container-id}-json.log
   ```
   Replace `{container-id}` with the ID of your container. This command will clear the logs for the specified container. Be careful, this method will completely remove the logs.

Remember to take into account the needs of your application and your capacity to store logs. Setting an adequate log retention policy is key to both ensuring the availability of relevant log data and preventing your storage from being filled with logs.