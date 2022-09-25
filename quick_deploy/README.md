# Face Recognition Bootcamp Project using Milvus.

This demo uses [Milvus Vector Database](https://milvus.io/) for storing face embeddings & performing face similaity search based on those stored embeddings. Milvus is a vector similarity search engine, a tool that lets you quickly find the closest matching vector in a pool of billions of vectors.


The system architecture is as below:
<p align="center">
<img src="workflow.png" width = "500" height = "600" alt="system_arch" />
</p>

## Data Source

This demo uses the dataset of around 800k images consisting of 1100 Famous Celebrities and an Unknown class to classify unknown faces. All the images have been scraped from Google and contains no duplicate images. Each Celebrity class(folder) consists approximately 700-800 images and the Unknown class consists of 100k images.

> Note: You can also use other images for testing. This system supports the following formats: .jpg and .png.

## Local Deployment

### Requirements

- [Milvus](https://milvus.io/docs/v2.0.0/install_standalone-docker.md)
- [SQLite](https://hub.docker.com/r/mysql/mysql-server)
- [Python3](https://www.python.org/downloads/)
- [Docker](https://docs.docker.com/engine/install/)

## Option 1: Deploy with Docker Compose

The face recognition bootcamp system requires Milvus, MySQL, WebServer and WebClient services. We can start these containers with one click through [docker-compose.yaml](./docker-compose.yaml).

- Modify docker-compose.yaml to map your data directory to the docker container of WebServer
```bash
$ git clone https://github.com/milvus-io/bootcamp.git
$ cd solutions/reverse_image_search/quick_deploy
$ vim docker-compose.yaml
```
> Change line 32: `./standalone:/image` --> `milvusdb:/your_milvus_version`

- Create containers & start servers with docker-compose.yaml
```bash
$ docker-compose up -d
```

Then you will see the that all containers are created.

```bash
[+] Running 12/12
 ⠿ standalone Pulled                                                                                                                                                                          28.5s
   ⠿ 171857c49d0f Pull complete                                                                                                                                                                4.6s
   ⠿ 419640447d26 Pull complete                                                                                                                                                                4.9s
   ⠿ 61e52f862619 Pull complete                                                                                                                                                                5.0s
   ⠿ 2580b47486e5 Pull complete                                                                                                                                                               20.3s
   ⠿ cd742921730d Pull complete                                                                                                                                                               22.2s
   ⠿ 936cb7027fe4 Pull complete                                                                                                                                                               22.2s
   ⠿ 319dd389c04d Pull complete                                                                                                                                                               24.2s
   ⠿ 543c11caaeb6 Pull complete                                                                                                                                                               24.3s
   ⠿ 06d62b89360c Pull complete                                                                                                                                                               24.5s
   ⠿ 5186d5863148 Pull complete                                                                                                                                                               24.6s
   ⠿ b410b80e82c0 Pull complete                                                                                                                                                               24.7s
[+] Running 3/3
 ⠿ Container milvus-minio       Started                                                                                                                                                        1.3s
 ⠿ Container milvus-etcd        Started                                                                                                                                                        1.3s
 ⠿ Container milvus-standalone  Started  
```

And show all containers with `docker ps`, and you can use `docker logs img-search-webserver` to get the logs of **server** container.

```bash
CONTAINER ID   IMAGE                                      COMMAND                  CREATED         STATUS                   PORTS                                           NAMES
3c8d3ceab1ab   milvusdb/milvus:v2.0.2                     "/tini -- milvus run…"   4 minutes ago   Up 4 minutes             0.0.0.0:19530->19530/tcp, :::19530->19530/tcp   milvus-standalone
43c1353b98a2   minio/minio:RELEASE.2020-12-03T00-03-10Z   "/usr/bin/docker-ent…"   4 minutes ago   Up 4 minutes (healthy)   9000/tcp                                        milvus-minio
ab2c1c06ea1e   quay.io/coreos/etcd:v3.5.0                 "etcd -advertise-cli…"   4 minutes ago   Up 4 minutes             2379-2380/tcp                                   milvus-etcd
```


## Option 2: Deploy with source code

We recommend using Docker Compose to deploy the face recognition bootcamp. However, you also can run from source code, you need to manually start [Milvus](https://milvus.io/docs/v2.0.0/install_standalone-docker.md) and [SQLite](https://dev.mysql.com/doc/mysql-installation-excerpt/5.7/en/docker-mysql-getting-started.html). Next show you how to run the API server and Client.

### 1. Start Milvus

First, you need to start Milvus & SQLite servers.

Refer [Milvus Standalone](https://milvus.io/docs/v2.0.0/install_standalone-docker.md) for how to install Milvus. Please note the Milvus version should match pymilvus version in [config.py](./server/src/config.py).

There are several ways to start Mysql. One option is using docker to create a container:

- Download Configuration Files

```bash
$ mkdir -p /home/$USER/milvus/conf
$ cd /home/$USER/milvus/conf
$ wget https://raw.githubusercontent.com/Spnetic-5/ospp_face_rec_draft/edit/main/quick_deploy/server_config.yaml
```


```bash
$ sudo docker run -d --name milvus_cpu_1.1.0 -p 19530:19530 -p 19121:19121 -v /home/$USER/milvus/db:/var/lib/milvus/db -v /home/$USER/milvus/conf:/var/lib/milvus/conf -v /home/$USER/milvus/logs:/var/lib/milvus/logs -v /home/$USER/milvus/wal:/var/lib/milvus/wal milvusdb/milvus:1.1.0-cpu-d050721-5e559c
```


### 2. Start API Server

Then to start the system server, and it provides HTTP backend services.

- **Install the Python packages**

```bash
$ git clone https://github.com/milvus-io/bootcamp.git
$ pip install -m requirements.txt
```

- **Set configuration**

Modify the parameters according to your own environment. Here listing some parameters that need to be set, for more information please refer to [config.py](./server/src/config.py).

| **Parameter**    | **Description**                                       | **Default setting** |
| ---------------- | ----------------------------------------------------- | ------------------- |
| MILVUS_HOST      | The IP address of Milvus, you can get it by ifconfig. | 127.0.0.1           |
| MILVUS_PORT      | Port of Milvus.                                       | 19530               |
| VECTOR_DIMENSION | Dimension of the vectors.                             | 1000                |
| MYSQL_HOST       | The IP address of Mysql.                              | 127.0.0.1           |
| MYSQL_PORT       | Port of Mysql.                                        | 3306                |
| DEFAULT_TABLE    | The milvus and mysql default collection name.         | milvus_img_search   |

 **Prepare the dataset for Milvus Search Engine**

```bash
python3 prepare_data.py
```

- **Run the code**

Then start the server.
- Replace test.jpg with <image_path>
```bash
python3 celeb_finder.py test.jpg
```

## Code  structure

If you are interested in our code or would like to contribute code, feel free to learn more about our code structure.

```bash
server
├── Dockerfile
├── requirements.txt
└── src
    ├── __init__.py
    ├── config.py # Configuration file
    ├── encode.py # Convert an image to embedding using towhee pipeline (ResNet50)
    ├── encode_tf_resnet50.py # Old encoder file using ResNet50 by tensorflow
    ├── logs.py
    ├── main.py # Source code to start webserver
    ├── milvus_helpers.py # Connect to Milvus server and insert/drop/query vectors in Milvus.
    ├── mysql_helpers.py # Connect to MySQL server, and add/delete/query IDs and object information.
    ├── operations
    │   ├── __init__.py
    │   ├── count.py
    │   ├── drop.py
    │   ├── load.py
    │   ├── search.py
    │   └── upload.py
    └── test_main.py # Pytest file for main.py
```

![Screenshot from 2022-08-13 01-57-52](https://user-images.githubusercontent.com/66636289/184447727-ec77dc47-25f7-430b-8593-1178683358f0.png)
