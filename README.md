## docker-encfs-rclone-mount

using an existing rclone.conf or setting up a new remote, you can run this container to mount your remote filesystem and share it with the host and other containers

Optionally, if a .encfs6.xml file exists in the mount point, it will be decrypted to /data-decrypted

* [rclone](https://rclone.org)
* [encfs](https://github.com/vgough/encfs)

shamelessly coppied from [tynor88](https://github.com/tynor88/docker-rclone-mount)

## Configuring rclone.conf

### Copying existing conf
	mkdir p $(pwd)/config/encfs-rclone-mount
	cp ~/.config/rclone/rclone.conf $(pwd)/config/encfs-rclone-mount/.rclone.conf
        cp ~/.encfs6.xml $(pwd)/config/encfs-rclone-mount/.encfs6.xml

### Creating a new conf
	docker run -it --rm \
		-v $(pwd)/config/encfs-rclone-mount:/config \
		pknw1/encfs-rclone-mount rclone --config="/config/.rclone.conf" config

## Running a configured instance
### docker-cli

	REMOTE=yourremotename
	docker run -d \
		--name=encfs-rclone-$REMOTE \
		-v $(pwd)/config/encfs-rclone-$REMOTE:/config \
		--cap-add SYS_ADMIN \
		--device /dev/fuse \
		--security-opt apparmor:unconfined \
		-e RCLONE_REMOTE_MOUNT=$REMOTE:/ \
		-e PUID=666 \
		-e PGID=666 \
		-e TZ=Europe/London \
		-v $(pwd)/config/encfs-rclone-$REMOTE:/config \
		-v /mnt/rclone-$REMOTE:/data:shared \
                -v /mnt/encfs-rclone-$REMOTE:/data-decrypted:shared \
		-v /etc/localtime:/etc/localtime \
		-v /tmp:/tmp \
		--restart=unless-stopped \
		pknw1/encfs-rclone-mount

### docker-compose

	version: '3'
	services:
	  rclone-ovh:
	    image: pknw1/encfs-rclone-mount
	    container_name: encfs-rclone-mount
	    security_opt:
	      - apparmor:unconfined
	    cap_add:
	      - SYS_ADMIN
	    devices:
	      - "/dev/fuse:/dev/fuse"
	    environment:
	      - PUID=666
	      - PGID=666
	      - PUMASK=0000
	      - TZ=Europe/London
	      - RCLONE_REMOTE_MOUNT=remote:/
	    volumes:
	      - ./config/encfs-rclone-mount:/config
	      - /mnt/rclone-remote:/data:shared
              - /mnt/encfs-rclone-remote:/data-decrypted:shared
	      - /tmp:/tmp
	      - /etc/localtime:/etc/localtime
	    restart: unless-stopped

## Further development

* encfs mounting
* automatically recognise encrypted volumes and mount depending on ENCFS_PASS environment variable
