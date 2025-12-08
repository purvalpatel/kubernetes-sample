Join Node in existing kubernetes cluster:
-------------------------------------
1. Install all kubernetes packages. ( On Worker Node)
 
2. Create New token: ( On Master node )
```
kubeadm token create --print-join-command
```
- This token will be expired after **24 hours**.
- Creating new token will be **safe**.
- If you want to add new node then you have to **generate new token.**
- Only used for **onboarding new workers**
- Recommended practice is to **generate a new token each time**

3. Paste the join command on Worker node.
