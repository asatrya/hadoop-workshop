# Working with Object Storage

## Introduction

In this workshop we will work with Object Storage for persistence, which can either be [MinIO](https://min.io/) as shown in the workshop, [Amazon S3](https://aws.amazon.com/s3/) or any other cloud Object Storage solution. We will use object storage as a drop-in replacement for Hadoop HDFS.

We assume that the **Analytics platform** described [here](../01-environment) is running and accessible. 

```
  minio:
    image: minio/minio
    container_name: minio
    hostname: minio
    ports:
      - '9000:9000'
    volumes:
      - './container-volume/minio/data/:/data'
#      - './minio/config:/root/.minio'
    environment:
      MINIO_ACCESS_KEY: V42FCGRVMK24JJ8DHUYG
      MINIO_SECRET_KEY: bKhWxVF3kQoLY9kFmt91l+tDrEoZjqnWXzY9Eza
    command: server /data
    restart: always
```


```
  awscli:
    image: xueshanf/awscli
    container_name: awscli
    hostname: awscli
    volumes:
      - './conf/s3cmd.config:/root/.s3cfg'
      - './data-transfer:/data-transfer'
    environment:
      AWS_ACCESS_KEY_ID: V42FCGRVMK24JJ8DHUYG
      AWS_SECRET_ACCESS_KEY: bKhWxVF3kQoLY9kFmt91l+tDrEoZjqnWXzY9Eza
    command: tail -f /dev/null
    restart: always
```

##	 Accessing MinIO

[MinIO](https://min.io/) is an object storage server released under Apache License v2.0. It is compatible with Amazon S3 cloud storage service. It is best suited for storing unstructured data such as photos, videos, log files, backups and container / VM images. Size of an object can range from a few KBs to a maximum of 5TB.

There are various ways for accessing MinIO

 * **S3cmd** - a command line S3 client for working with S3 compliant object stores
 * **MinIO UI** - a browser based GUI for working with MinIO
 * **Apache Zeppelin** - a browser based GUI for working with various tools of the Big Data ecosystem

These are only a few of the tools available to work with S3. And because an Object Store is in fact a drop-in replacement for HDFS, we can also use it from the tools in the Big Data ecosystem such as Hadoop Hive, Spark, ...

### Using S3cmd

[S3cmd](https://s3tools.org/s3cmd) is a command line utility for working with S3. 

In our environment, S3cmd is accessible inside the `awscli`container.

Go to container's Bash shell

```
bash-5.0# bash
```

Running `s3cmd -h` will show the help page of s3cmd.

```
bash-5.0# s3cmd -h
```

This can also be found on the [S3cmd usage page](https://s3tools.org/usage).

### Using MinIO UI

In a browser window, navigate to <http://localhost:9000> and you should see login screen. Enter `V42FCGRVMK24JJ8DHUYG` into the **Access Key** and  `bKhWxVF3kQoLY9kFmt91l+tDrEoZjqnWXzY9Eza` into the **Secret Key** field and click on the connect button.  
The MinIO homepage should now appear.
 
![Alt Image Text](./images/minio-home.png "Minio Homepage")

Click on the **+** icon at the lower right corner to either perform an **Create bucket** or **Upload file** action.

![Alt Image Text](./images/minio-action-menu.png "Minio Action Menu")

##	 Moving data to Object Storage

Let's upload the files of the Hive workshop to the object storage. 

First we have to create a bucket.

### Create a Bucket

Here are the commands to perform when using the **S3cmd** on the command line

```
bash-5.0# s3cmd mb s3://truck-bucket
```

and you should get the bucket created method as shown below

```
bash-5.0# s3cmd mb s3://truck-bucket
Bucket 's3://truck-bucket/' created
```

Navigate to the MinIO UI (<http://localhost:9000>) and you should see the newly created bucket in the left. 

![Alt Image Text](./images/minio-show-bucket.png "Minio show bucket")

or you could also use `s3cmd ls` to list all buckets.

```
bash-5.0# s3cmd ls s3://
```

### Upload the files to the bucket

To upload a file we are going to use the `s3cmd put` command. First for the `trucks.csv`

```
bash-5.0# s3cmd put /data-transfer/truckdata/trucks.csv s3://truck-bucket/raw/trucks.csv
```

and then also for the `geolocation.csv` file. 

```
bash-5.0# s3cmd put /data-transfer/truckdata/geolocation.csv s3://truck-bucket/raw/geolocation.csv
```

Let's use the `s3cmd ls` command once more but now to display the content of the `truck-bucket`

```
bash-5.0# s3cmd ls s3://truck-bucket/
```

We can see that the bucket contains a directory with the name `raw`, which is the prefix we have used when uploading the data above. 

```
bash-5.0# s3cmd ls s3://truck-bucket/
                       DIR   s3://truck-bucket/raw/
```

If we use the `-r` argument

```
bash-5.0# s3cmd ls -r  s3://truck-bucket/
```

we can see the objects with the hierarchy as well. 

```
bash-5.0# s3cmd ls -r  s3://truck-bucket/
2019-05-27 13:39    526677   s3://truck-bucket/raw/geolocation.csv
2019-05-27 13:37     61378   s3://truck-bucket/raw/trucks.csv
```

We can see the same in the MinioUI.  

![Alt Image Text](./images/minio-list-objects.png "Minio list objects")


Let's upload file in JSON format

```
bash-5.0# s3cmd put /data-transfer/truckdata/truck_mileage.json s3://truck-bucket/result/json/truck_mileage.json
```

We use the prefix `result` to separate the files from the `raw` files uploaded previously.
Next let's do the same for the parquet

```
bash-5.0# s3cmd put /data-transfer/truckdata/truck_mileage.parquet s3://truck-bucket/result/parquet/truck_mileage.parquet
```

and for the ORC file. 

```
bash-5.0# s3cmd put /data-transfer/truckdata/truck_mileage.orc s3://truck-bucket/result/orc/truck_mileage.orc
```

Let's again show the objects using the `s3cmd ls -r s3://truck-bucket` command:

```
bash-5.0# s3cmd ls -r s3://truck-bucket/
2019-05-28 11:17    526677   s3://truck-bucket/raw/geolocation.csv
2019-05-28 11:15     61378   s3://truck-bucket/raw/trucks.csv
2019-05-28 11:18    537544   s3://truck-bucket/result/json/truck_mileage.json
2019-05-28 11:18     62377   s3://truck-bucket/result/orc/truck_mileage.orc
2019-05-28 11:18     76854   s3://truck-bucket/result/parquet/truck_mileage.parquet
```

We have successfully uploaded the necessary files as object into the Object Storage.