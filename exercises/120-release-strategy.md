#### 1. Create a Deployment manifest which uses `Recreate` as its deployment strategy with the following properties.
```
Name: mogambo
Replicas: 3
Image: schoolofdevops/frontend:latest
```

#### 2. Create a Deployment with version 1 of the image and then do a rolling update to version 2.
```
Image Version 1: schoolofdevops/frontend:v1.0
Image Version 2: schoolofdevops/frontend:v2.0
maxUnavailable pods should be 1.
maxSurge should be 2.
```

#### 3. Convert the above created Deployment manifest with Blue/Green strategy.
```
Version 1: Blue
Version 2: Green
```

