# Kubernetes Note 

### 1. Kiến trúc của Kubernetes
- Kubernetes cluster bao gồm 2 thành phần chính sau:
  + Master nodes
  + Worker notes
- Master node bao gồm 4 thành phần chính là : API Manager, Controller Manager, Scheduler và Etcd
  + Api Server : Thành phần chính giao tiếp với các thành phần khác
  + Controller Manager : gồm nhiều controller riêng cụ thể cho từng resource và thực hiện các chứng năng cụ thể cho từng thằng resource trong kube như create pod, create deployment
  + Scheduler: schedules ứng dụng tới node nào
  + Etcd: là một database để lưu giữ trạng thái và resource của cluster


### 2. Tự động tạo và quản lý pod : ReplicationControllers , ReplicaSets
- ReplicationControllers:
  + Là Một resource mà sẽ tạo và quản lý pod, đảm bảo số lượng pod luôn running
  + Bằng cách tạo ra số lượng pod bằng với giá trị chỉ định ở thuộc tính "replica", và quản lý thông qua labels của pod
  ![1.png](img_guide/1.png)
  + Ta biết rằng Pod giám sát container và tự động restart lại container khi nó fail
  + Trong trường hợp toàn bộ worker node fail thì pod sẽ ko chạy nữa ?
  + Nếu chạy cluster với nhiều hơn 1 Worker node, Th ReplicationControllers sẽ giúp giải quyết vấn đề trên
  + Replication Controller nếu phát hiện ra số lượng pod của một worker_node nào đó = 0 -> Sẽ tạo một Pod ở worker node khác để
  + Ngoài ra, Replication Controller còn giúp tăng performance của ứng dụng bằng cách chỉ định số lượng replicas trong Replcation Controller để tạo ra nhiều pod chạy một ứng dụng
  + Ví dụ ta có một webservice, nếu ta chỉ deploy một pod để chạy ứng dụng, thì ta chỉ có 1 container để xử lý request của user, nhưng nếu ta dùng RC và chỉ định replicas = 3, ta sẽ có 3 pod chạy 3 container của ứng dụng, và request của user sẽ được gửi tới 1 trong 3 pod này, giúp quá trình xử lý của chúng ta tăng gấp 3 lần
  ![2.png](img_guide/2.png)


### 3. Ví dụ tạo Pod
- Source code : [example_pod](/example_pod)
- Tạo file index.js
- Tạo file Dockerfile
- Tạo file hello-kube.yml
- 




