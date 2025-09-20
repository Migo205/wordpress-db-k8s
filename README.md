
# WordPress on Kubernetes with AWS: A Cloud-Native Deployment Guide 🚀

### Project Overview 🌐

This repository contains all the necessary **Kubernetes manifest files** to deploy a complete, production-ready **WordPress** application and its associated **MySQL** database. The entire stack is built to run on a **Kubernetes cluster** while utilizing robust and scalable storage solutions from **AWS (Amazon Web Services)**. This project serves as an excellent example של a real-world, cloud-native application deployment, showcasing key **DevOps** and cloud engineering principles.

-----

### Understanding the Project Architecture 🏗️

The core philosophy of this project is **separation of concerns**. Instead of running the entire application on a single server, we break it down into independent, manageable services. This design makes the application more resilient, scalable, and easier to maintain.

  * **MySQL Service:** This is our database layer. A **MySQL Deployment** manages the **MySQL container** to ensure it's always running. This service is connected to a **Persistent Volume (PV)**, which is like a virtual hard drive on AWS. This PV is backed by **EBS (Elastic Block Store)**, a block storage solution that provides high-performance, dedicated storage. The database credentials are kept secure in a **Kubernetes Secret**, which the MySQL container uses.
  * **WordPress Service:** This is the application layer that users interact with. A **WordPress Deployment** manages the container and connects to the MySQL service using a stable internal address provided by a **Kubernetes Service**. For file storage, it uses a separate **Persistent Volume** backed by **EFS (Elastic File System)**. Unlike EBS, EFS is a **shared file system**, allowing multiple WordPress pods to read from and write to the same location at the same time.
  * **Kubernetes Services:** The `Service` objects act as a stable **network gateway**. They provide a single, unchanging IP address and DNS name for the pods they manage. This allows the WordPress application to find the MySQL database easily, and it allows users to access the WordPress site even if the underlying pods' addresses change.

-----

### Key Kubernetes Concepts Explained 📖

This project utilizes several fundamental Kubernetes resources. Here's a quick rundown of their purpose:

  * **Deployment:** A **Deployment** manages a set of identical pods. Its main job is to ensure the desired number of replicas of your application are running at all times. It also handles rolling updates and rollbacks, making it easy to deploy new versions of your application without downtime.
  * **Service:** A **Service** provides a stable network endpoint for a set of pods. It acts as an internal load balancer and a DNS entry, abstracting away the dynamic IP addresses of the pods. This is what enables the WordPress deployment to find the MySQL deployment without needing to know its exact location.
  * **PersistentVolume (PV) & PersistentVolumeClaim (PVC):** This is Kubernetes' solution for **persistent storage**. The **PV** is the "physical" storage (like the hard drive on AWS), created by the administrator. The **PVC** is a "request" for that storage, made by the application. This separation allows developers to request storage without needing to know the technical details of the underlying hardware.
  * **StorageClass:** A **StorageClass** is a blueprint that defines a type of storage. It enables **dynamic provisioning**, where Kubernetes can automatically create the underlying PV based on a PVC's request.
  * **Secret:** A **Secret** is a resource used to securely store sensitive data like passwords, API keys, and tokens. Storing this information as a Secret prevents it from being exposed in your application's source code.

-----

### Deployment Prerequisites ⚙️

Before you begin, ensure you have the following:

  * **`kubectl`:** The command-line tool for interacting with your Kubernetes cluster.
  * **A running Kubernetes cluster on AWS:** Your cluster must have the necessary **CSI drivers** for both EBS and EFS installed.
  * **Manual EFS setup:** Since the EFS driver does not support dynamic provisioning, you must manually create an **EFS File System** on your AWS account. You will also need to create an **Access Point** within that file system and update the `wordpress-pv.yml` file with the correct **File System ID** and **Access Point ID**.

-----

### Deployment Steps 🚀

Follow these steps in the exact order to deploy the application successfully:

1.  **Deploy Secrets & Storage:**
    ```bash
    # Create the secret for the MySQL database credentials.
    kubectl apply -f mysql-secret.yml

    # Deploy storage components for both services.
    # Note: A PV is manually created for WordPress to link to the pre-existing EFS file system.
    kubectl apply -f mysql-sc.yml
    kubectl apply -f mysql-pvc.yml
    kubectl apply -f wordpress-sc.yml
    kubectl apply -f wordpress-pv.yml
    kubectl apply -f wordpress-pvc.yml
    ```
2.  **Deploy Applications & Services:**
    ```bash
    # Deploy the MySQL service and application.
    kubectl apply -f mysql-app.yml
    kubectl apply -f mysql-svc.yml

    # Deploy the WordPress service and application.
    kubectl apply -f wordpress-app.yml
    kubectl apply -f wordpress-svc.yml
    ```

-----

### Accessing the Site 💻

Once all the components are deployed, it will take a few moments for the `LoadBalancer` to provision. You can track its status by checking the services:

```bash
kubectl get svc wordpress-svc
```

When the `EXTERNAL-IP` field is no longer `<pending>`, you can copy the IP address and paste it into your web browser to access your new WordPress site\!
