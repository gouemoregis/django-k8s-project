1. Test Django
```
python manage.py test
````

2. Build container
```
docker build -f Dockerfile \
    -t registry.digitalocean.com/regis-k8s/django-k8s-web:latest \
    -t registry.digitalocean.com/regis-k8s/django-k8s-web:v1 \
    .
```

3. Push this container to DO Container Registry with 2 tags: latest and random
```
docker push registry.digitalocean.com/regis-k8s/django-k8s-web --all-tags
```

4. Update secrets (if needed)
```
kubectl delete secret django-k8s-web-prod-env
kubectl create secret generic django-k8s-web-prod-env --from-env-file=web/.env.prod
```

5. Update Deployment
```
kubectl apply -f k8s/apps/django-k8s-web.yaml
```

6. Wait for Rollout to finish
```
kubectl rollout status deployment/django-k8s-web-deployment
```

7. Migrate Database
```
export SINGLE_POD_NAME=$(kubectl get pod -l app=django-k8s-web-deployment -o jsonpath="{.items[0].metadata.name}")
```
or
```
export SINGLE_POD_NAME=$(kubectl get pod -l app=django-k8s-web-deployment -o NAME | tail -n 1)
```

Run the migrations
```
kubectl exec -it $SINGLE_POD_NAME -- bash /app/migrate.sh
```

