# devops-demo: Java CI/CD on AWS EKS

GitHub → GitHub Actions (Gitleaks, Maven build+test) → GHCR image → AWS EKS (kubectl rollout)

## One-time setup
1. Create an empty GitHub repo `akileshsekar2005/devops-demo`, then in this folder:
   `rm -rf .git && git init -b main && git add . && git commit -m "Initial commit" && git remote add origin https://github.com/akileshsekar2005/devops-demo.git && git push -u origin main`
   (the first run will fail until steps 2-4 are done; that's expected)
2. Create the cluster: `eksctl create cluster -f eks-cluster.yaml` (region ap-south-1)
3. In AWS IAM: add the GitHub OIDC identity provider (`token.actions.githubusercontent.com`, audience `sts.amazonaws.com`) and create a role trusted for `repo:akileshsekar2005/devops-demo:ref:refs/heads/main`. Give it `eks:DescribeCluster`, then map it into the cluster:
   `eksctl create iamidentitymapping --cluster devops-demo-cluster --region ap-south-1 --arn <ROLE_ARN> --username github-actions --group system:masters`
4. GitHub repo → Settings → Secrets and variables → Actions → add secret `AWS_ROLE_ARN` = the role ARN.
5. After the first image is pushed, GitHub profile → Packages → `devops-demo` → Package settings → set visibility **Public** (otherwise EKS cannot pull it).

## Deploy / redeploy
Push to `main`. Get the URL with `kubectl get svc devops-demo-service` and open `http://<EXTERNAL-IP>/health`.

## Clean up
`kubectl delete -f k8s/` then `eksctl delete cluster -f eks-cluster.yaml`

`Jenkinsfile` and `ansible/` are leftovers from an older EC2 version and are not used by this pipeline.
