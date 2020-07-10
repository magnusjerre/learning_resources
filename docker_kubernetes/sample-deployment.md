apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubernetes-bootcamp-script
  labels:
    app: kubernetes-bootcamp-script
spec:
  selector:
    matchLabels:
      app: kubernetes-bootcamp-script
  replicas: 1
  template:
    metadata:
      labels:
        app: kubernetes-bootcamp-script
    spec:
      containers:
        - name: master
          image: gcr.io/google-samples/kubernetes-bootcamp:v1
          ports:
          - containerPort: 8080