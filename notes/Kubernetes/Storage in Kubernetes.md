# Storage in Kubernetes

## Storge in Docker:

- **Docker File system in host:**

  - by default docker stores data in host at
  - Images are stored in images folder
  - container data is stored in container folder
  - volumes in volumes folder
    ```shell
    /var/lib/docker
        aufs
        containers
        image
        volumes
    ```

### **Docker layer archetrcutre:**
  - each instruction in docker file has its own layer
  - If the some of the layers is already used by the some other docker file previously then docker will not create the same layer again it will reuse it
    - this will save Docker image build time
    - this will save Docker image size
### **Type of layers in docker**
  - _Image layer_
    - This is a `readOnly` layer, where user cannot modify the containts of it.
    - if you want to modify let say application source code then docker will automatically creates the copy of it from the image layer to the container layer where you can be able to modify it.
  - _Container layer_
    - This layer can be modify `read-write` by the user
    - container layer is temp layer and only created when the container is created and deleted when the container is bing deleted.
  - Once the image is created its layers are been shared by other images too.
  - Once the container is deleted the data which is been changed also been deleted, to persists data we need to use the docker volmes.
### **Docker Volumes**
  - _Volume Mount_
    - with volume mount you can mount the volume in host with inside the file directory in docker container
    - Even if the container gets deleted the data still remains inside the volume in the host.
      ```
      docker create volume volume1
      docker run -v volume1:/app myimage
      ```
  - _Bind Mount_
    - Now suppose you want to mount a directory of host to the directory of container you can achive this with the help of Bind mount
      ```
      docker run -v /temp/app:/app myimage
      docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
      ```
### **Storage Drivers**
  - Docker automatically choose the Storage drive based on OS.
  - Docker supports multiple storage drivers
    - AUFS
    - ZFS
    - BTRFS
    - Device Mapper
    - Overlay
    - Overlay2

---

## Docker Volumes

- **Docker Volumes** are used to persist data even after the container is deleted.
- You can use the Volume Drive to presist the data to AWS, GCP or Azure.
  - local
  - Azure file Storage
  - Convoy
  - Digital Ocean Block Storage
  - NetApp
  - etc.
  ```
  docker run -it --name mysql --volume-driver rexray/ebs --mount src=ebs-vol,target=/var/lib/mysql mysql
  ```
  - this will provsion a AWS EBS to store the data from the Docker container volume to Cloud.

## Kubernetes interfaces standards for any container technology:

- CRI - Container Runtime Interface
- CNI - Container Network Interface
- CSI - Container Storage Interface

## PersistantVolume (Refer above)

## PersistentVolumeClaim (Refer above)
