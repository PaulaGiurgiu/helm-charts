# helm-chart-repo

Commands:
- helm create webapp
- helm repo index --url https://github.com/PaulaGiurgiu/helm-charts.git .
- helm install webapp ./webapp
NAME: webapp
LAST DEPLOYED: Fri Jun  7 11:31:13 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=webapp,app.kubernetes.io/instance=webapp" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT

- kubectl get secrets 
- kubectl get secret -o yaml  
- kubectl get secrets/sh.helm.release.v1.webapp.v1 -o json | jq -r .data.release | base64 -d | base64 -d | gunzip -c | jq -r '.chart.template[].data' | base64 -d
- helm repo add helm-charts https://github.com/PaulaGiurgiu/helm-charts.git .

- helm install release1 . (/webapp)


# old deployment: 

# apiVersion: apps/v1
# kind: Deployment
# metadata:
#   name: {{ include "webapp.fullname" . }}
#   labels:
#     {{- include "webapp.labels" . | nindent 4 }}
# spec:
#   {{- if not .Values.autoscaling.enabled }}
#   replicas: {{ .Values.replicaCount }}
#   {{- end }}
#   selector:
#     matchLabels:
#       {{- include "webapp.selectorLabels" . | nindent 6 }}
#   template:
#     metadata:
#       {{- with .Values.podAnnotations }}
#       annotations:
#         {{- toYaml . | nindent 8 }}
#       {{- end }}
#       labels:
#         {{- include "webapp.labels" . | nindent 8 }}
#         {{- with .Values.podLabels }}
#         {{- toYaml . | nindent 8 }}
#         {{- end }}
#     spec:
#       {{- with .Values.imagePullSecrets }}
#       imagePullSecrets:
#         {{- toYaml . | nindent 8 }}
#       {{- end }}
#       serviceAccountName: {{ include "webapp.serviceAccountName" . }}
#       securityContext:
#         {{- toYaml .Values.podSecurityContext | nindent 8 }}
#       containers:
#         - name: {{ .Chart.Name }}
#           securityContext:
#             {{- toYaml .Values.securityContext | nindent 12 }}
#           image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
#           imagePullPolicy: {{ .Values.image.pullPolicy }}
#           ports:
#             - name: http
#               containerPort: {{ .Values.service.port }}
#               protocol: TCP
#           livenessProbe:
#             {{- toYaml .Values.livenessProbe | nindent 12 }}
#           readinessProbe:
#             {{- toYaml .Values.readinessProbe | nindent 12 }}
#           resources:
#             {{- toYaml .Values.resources | nindent 12 }}
#           {{- with .Values.volumeMounts }}
#           volumeMounts:
#             {{- toYaml . | nindent 12 }}
#           {{- end }}
#       {{- with .Values.volumes }}
#       volumes:
#         {{- toYaml . | nindent 8 }}
#       {{- end }}
#       {{- with .Values.nodeSelector }}
#       nodeSelector:
#         {{- toYaml . | nindent 8 }}
#       {{- end }}
#       {{- with .Values.affinity }}
#       affinity:
#         {{- toYaml . | nindent 8 }}
#       {{- end }}
#       {{- with .Values.tolerations }}
#       tolerations:
#         {{- toYaml . | nindent 8 }}
#       {{- end }}



# odl services:
     # targetPort: 8080
      

# apiVersion: v1
# kind: Service
# metadata:
#   name: {{ include "webapp.fullname" . }}
#   labels:
#     {{- include "webapp.labels" . | nindent 4 }}
# spec:
#   type: {{ .Values.service.type }}
#   ports:
#     - port: {{ .Values.service.port }}
#       targetPort: http
#       protocol: TCP
#       name: http
#   selector:
#     {{- include "webapp.selectorLabels" . | nindent 4 }}



# Ingress Example

# {{- if .Values.ingress.enabled -}}
# {{- $fullName := include "webapp.fullname" . -}}
# {{- $svcPort := .Values.service.port -}}
# {{- if and .Values.ingress.className (not (semverCompare ">=1.18-0" .Capabilities.KubeVersion.GitVersion)) }}
#   {{- if not (hasKey .Values.ingress.annotations "kubernetes.io/ingress.class") }}
#   {{- $_ := set .Values.ingress.annotations "kubernetes.io/ingress.class" .Values.ingress.className}}
#   {{- end }}
# {{- end }}
# {{- if semverCompare ">=1.19-0" .Capabilities.KubeVersion.GitVersion -}}
# apiVersion: networking.k8s.io/v1
# {{- else if semverCompare ">=1.14-0" .Capabilities.KubeVersion.GitVersion -}}
# apiVersion: networking.k8s.io/v1beta1
# {{- else -}}
# apiVersion: extensions/v1beta1
# {{- end }}
# kind: Ingress
# metadata:
#   name: {{ $fullName }}
#   labels:
#     {{- include "webapp.labels" . | nindent 4 }}
#   {{- with .Values.ingress.annotations }}
#   annotations:
#     {{- toYaml . | nindent 4 }}
#   {{- end }}
# spec:
#   {{- if and .Values.ingress.className (semverCompare ">=1.18-0" .Capabilities.KubeVersion.GitVersion) }}
#   ingressClassName: {{ .Values.ingress.className }}
#   {{- end }}
#   {{- if .Values.ingress.tls }}
#   tls:
#     {{- range .Values.ingress.tls }}
#     - hosts:
#         {{- range .hosts }}
#         - {{ . | quote }}
#         {{- end }}
#       secretName: {{ .secretName }}
#     {{- end }}
#   {{- end }}
#   rules:
#     {{- range .Values.ingress.hosts }}
#     - host: {{ .host | quote }}
#       http:
#         paths:
#           {{- range .paths }}
#           - path: {{ .path }}
#             {{- if and .pathType (semverCompare ">=1.18-0" $.Capabilities.KubeVersion.GitVersion) }}
#             pathType: {{ .pathType }}
#             {{- end }}
#             backend:
#               {{- if semverCompare ">=1.19-0" $.Capabilities.KubeVersion.GitVersion }}
#               service:
#                 name: {{ $fullName }}
#                 port:
#                   number: {{ $svcPort }}
#               {{- else }}
#               serviceName: {{ $fullName }}
#               servicePort: {{ $svcPort }}
#               {{- end }}
#           {{- end }}
#     {{- end }}
# {{- end }}


 helm lint webapp
==> Linting webapp
[INFO] Chart.yaml: icon is recommended
1 chart(s) linted, 0 chart(s) failed
==> Linting webapp
1 chart(s) linted, 0 chart(s) failed
PS C:\Users\paula\Master\An1\Docker\helm-charts> helm package webapp\Chart.yaml
Error: Chart.yaml file is missing
PS C:\Users\paula\Master\An1\Docker\helm-charts> helm package webapp
Successfully packaged chart and saved it to: C:\Users\paula\Master\An1\Docker\helm-charts\webapp-0.1.0.tgz
PS C:\Users\paula\Master\An1\Docker\helm-charts> helm lint webapp
==> Linting webapp

1 chart(s) linted, 0 chart(s) failed
PS C:\Users\paula\Master\An1\Docker\helm-charts>


<!-- Important -->
helm show all webapp
apiVersion: v2
appVersion: 1.16.1
description: A Helm chart for Kubernetes
icon: https://media.istockphoto.com/id/1464561797/ro/fotografie/unitate-de-procesare-a-inteligen%C8%9Bei-artificiale-component%C4%83-puternic%C4%83-quantum-ai-pe-placa.jpg?s=1024x1024&w=is&k=20&c=rxIATtZuca76x9qX0F7-Zj3aCHREwiX2sHcvd_aL8LQ=
name: webapp
type: application
version: 0.1.0

---
namespace: webapp-namespace
<!--  -->

helm show values webapp    
namespace: webapp-namespace


<!--  -->
# helm install release1 webapp --namespace webapp-namespace
NAME: release1
LAST DEPLOYED: Fri Jun  7 15:13:22 2024
NAMESPACE: webapp-namespace
STATUS: deployed
REVISION: 1
TEST SUITE: None

# helm list -n webapp-namespace
NAME            NAMESPACE               REVISION        UPDATED                                 STATUS          CHART           APP VERSION
release1        webapp-namespace        1               2024-06-07 15:13:22.8930432 +0300 EEST  deployed        webapp-0.1.0

# helm package .
Successfully packaged chart and saved it to: C:\Users\paula\Master\An1\Docker\helm-charts\webapp\webapp-0.1.0.tgz

helm package . -d .. 
Successfully packaged chart and saved it to: ..\webapp-0.1.0.tgz


PS C:\Users\paula\Master\An1\Docker\helm-charts> tar xf .\webapp-0.1.0.tgz

helm repo index --url https://github.com/PaulaGiurgiu/helm-charts.git .