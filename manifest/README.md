kubectl exec frontend -- curl --trace - --trace-time backend 
kubectl exec backend -- curl --trace - --trace-time frontend 
kubectl exec frontend -- curl --trace - --trace-time database


kubectl -n option exec database -- curl --trace - --trace-time database.option
kubectl -n default exec backend -- curl --trace - --trace-time database.option