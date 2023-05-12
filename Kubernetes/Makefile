kubectl-get-all-deployments:
	kubectl get deployments

kubectl-apply-deployment:
	kubectl apply -f ./src/Deployment.yml

kubectl-delete-deployment:
	kubectl delete -f ./src/Deployment.yml



kubectl-get-pods:
	kubectl get pods

kubectl-exec-pod-multitool:
	kubectl exec -it multitool-busybox-869cbd767d-6cbwq -- cat /input/success.txt

kubectl-get-logs-pod:
	kubectl logs multitool-59qx6

kubectl-get-logs-daemon-set:
	kubectl logs daemonset/multitool


kubectl-get-daemon-set:
	kubectl get daemonset

kubectl-describe-daemon-set:
	kubectl describe daemonset/multitool

kubectl-apply-deployment2:
	kubectl apply -f ./src/Deployment2.yml

kubectl-delete-deployment2:
	kubectl delete -f ./src/Deployment2.yml

kubectl-exec-pod-multitool-2:
	kubectl exec -it multitool-4hjpl -- ls -la log