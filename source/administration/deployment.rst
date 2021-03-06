Deployment
----------

.. highlight:: yaml

Requirements
^^^^^^^^^^^^
You must have a Kubernetes cluster.

Platform deployment
^^^^^^^^^^^^^^^^^^^

.. note:: You can download a single Kubernetes Manifest containing all the necessary objects declarations :download:`here <../files/deployment.yaml>`.

.. warning:: This will expose the platform as a HTTP service. In production, you must deploy it in HTTPS!

First you need to clone the platform repository, and build the back and front
images::

    git clone adresse
    cd back
    docker build
    TODO: Finish these steps

Then, create a Kubernetes Deployment to deploy the back::

    apiVersion: apps/v1
    kind: Deployment
    metadata:
        name: back
    spec:
        replicas: 3
        selector:
            matchLabels:
                app: platform
                tier: backend
        template:
            metadata:
                labels:
                    app: platform
                    tier: backend
            spec:
                containers:
                  - name: back
                    image: back
                    imagePullPolicy: Never
                    ports:
                      - containerPort: 5000

.. note:: You should adapt the number of replicas to your needs.

Do the same for the front::

    apiVersion: apps/v1
    kind: Deployment
    metadata:
        name: front
    spec:
        replicas: 3
        selector:
            matchLabels:
                app: platform
                tier: frontend
        template:
            metadata:
                labels:
                    app: platform
                    tier: frontend
            spec:
                containers:
                  - name: front
                    image: front
                    imagePullPolicy: Never
                    ports:
                      - containerPort: 80

Now, create a Service to expose the backend::

    apiVersion: v1
    kind: Service
    metadata:
        name: back
    spec:
        selector:
            app: platform
            tier: backend
        ports:
          - protocol: TCP
            port: 5000
            targetPort: http

Now you can add a Service and an Ingress to expose the web server on the
network::

    apiVersion: v1
    kind: Service
    metadata:
        name: front
    spec:
        selector:
            app: platform
            tier: frontend
        ports:
          - protocol: TCP
            port: 80
    ---
    apiVersion: networking.k8s.io/v1beta1
    kind: Ingress
    metadata:
        name: front-ingress
        annotations:
            nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
        backend:
            serviceName: front
            servicePort: 80
