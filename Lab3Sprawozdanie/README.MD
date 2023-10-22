# Lab3 - Wielokontenerowy pod z sidecarem w Kubernetes

`Readme.md` zawiera opis rozwiązania problemu z utworzeniem wielokontenerowego poda w Kubernetes z użyciem sidecara.

## Opis Problemu

Zadaniem było utworzenie wielokontenerowego poda w przestrzeni nazw `lab3` w Kubernetes. Pod miał spełniać następujące założenia:

- W podstawowym kontenerze działa busybox i zapisuje dane wyjściowe za pomocą polecenia `date` do pliku `/var/log/date.log` co 5 sekund.

- Drugi kontener miał działać jako sidecar i zawierać serwer Nginx, który zapewnia dostęp do pliku `/var/log/date.log` przy użyciu woluminu współdzielonego typu hostPath (zamontowanego w `/usr/share/nginx/html`).

## Rozwiązanie

Rozwiązanie opiera się na utworzeniu pliku YAML (`Sprawozdanie.yaml`) dla poda w Kubernetes-ie.

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: lab3
spec: {}
status: {}
---
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-pod
  namespace: lab3
spec:
  containers:
    - name: busybox-container
      image: busybox
      command:
        [
          "/bin/sh",
          "-c",
          "while true; do date >> /var/log/date.log; sleep 5; done",
        ]
      volumeMounts:
        - name: log-volume
          mountPath: /var/log
    - name: nginx-container
      image: nginx
      volumeMounts:
        - name: log-volume
          mountPath: /usr/share/nginx/html
  volumes:
    - name: log-volume
      hostPath:
        path: /var/log
```

Kod tworzy poda o nazwie `sidecar-pod` w przestrzeni nazw `lab3`. Kontener `busybox-container` generuje dane co 5 sekund i zapisuje je do pliku `/var/log/date.log`. Kontener `nginx-container` działa jako sidecar i udostępnia plik `/var/log/date.log` przez serwer Nginx.

## Testowanie Rozwiązania

Aby potwierdzić poprawność działania poda, naley użyć port forwarding i narzędzia curl. Poniżej przedstawiono kroki do testowania:

1. Uruchomić port forwarding, aby uzyskać dostęp do serwera Nginx:
   kubectl port-forward -n lab3 sidecar-pod 8080:80 &

2. Uyć `curl`, aby uzyskać dostęp do pliku /var/log/date.log:
   curl http://localhost:8080/date.log

Spodziewane jest wyświetlenie wyników date co 5 sekund w pliku /var/log/date.log.
