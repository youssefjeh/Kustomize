### Comprendre Kustomize : Une Introduction Approfondie

Lorsque l'on travaille avec Kubernetes, la gestion des fichiers de configuration YAML peut rapidement devenir complexe, en particulier lorsqu'il s'agit de gérer plusieurs environnements (développement, production, etc.). Kustomize est un outil intégré à Kubernetes conçu pour répondre à ce besoin. Découvrons ensemble ce qu'est Kustomize, son fonctionnement, et comment il facilite la gestion des configurations Kubernetes.

---

### Qu'est-ce que Kustomize ?

Kustomize est un outil intégré à `kubectl`, ce qui signifie qu'il est directement disponible si vous utilisez Kubernetes. Contrairement à Helm, qui repose sur un système de *templating*, Kustomize est spécifiquement conçu pour la gestion des configurations sans dépendre de templates. Il permet de définir une configuration de base et de la personnaliser facilement pour différents environnements.

---

### Pourquoi utiliser Kustomize ?

1. **Gestion simplifiée des environnements** : Modifiez vos configurations de base en fonction des besoins spécifiques d'un environnement (dev, staging, production) sans dupliquer vos fichiers YAML.
2. **Évitez les duplications** : Les configurations partagées peuvent être définies une seule fois et réutilisées.
3. **Patchs flexibles** : Apportez des modifications spécifiques à certaines ressources Kubernetes sans remplacer intégralement les fichiers de configuration.

---

### Exemple de Configuration avec Kustomize

#### Structure de base

Imaginons une application ayant les fichiers suivants :
```
/kubernetes
    /application
        deployment.yaml
        configmap.yaml
```

#### Contenu des fichiers

**`deployment.yaml`**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 1
  template:
    containers:
      - name: my-container
        image: ubuntu:latest
        env:
        - name: TEST
          value: TOTO
        volumeMounts:
        - name: config-volume
          mountPath: /configs/
      volumes:
      - name: config-volume
        configMap:
          name: example-config
```

**`configmap.yaml`**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-config
  namespace: example
data:
  config.json: |
    {
      "environment": "test"
    }
```

---

#### Fichier `kustomization.yaml`

Ajoutez un fichier `kustomization.yaml` au dossier `/application` :
```yaml
resources:
  - deployment.yaml
  - configmap.yaml
```

Avec cette configuration, vous pouvez appliquer directement les ressources définies avec la commande suivante :
```bash
kubectl apply -k ./kubernetes/application
```

---

### Gestion des Environnements

Pour gérer plusieurs environnements (par exemple, développement et production), créez la structure suivante :
```
/kubernetes
    /environments
        /dev
            kustomization.yaml
        /prod
            kustomization.yaml
```

Dans chaque fichier `kustomization.yaml` des environnements, définissez la base :
```yaml
bases:
  - ../../application
```

Ainsi, appliquer la configuration pour un environnement est simple :
```bash
kubectl apply -k ./kubernetes/environments/dev
```

---

### Personnalisation Avancée

#### Ajouter des Patchs
Ajoutez des fichiers de patch pour modifier les configurations de base. Par exemple :

**`replica_count.yaml`**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 6
```

Dans `kustomization.yaml` :
```yaml
patches:
  - replica_count.yaml
```

#### Ajouter dynamiquement des variables d’environnement
Créez un fichier `env.yaml` :
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  template:
    containers:
      - name: my-container
        env:
        - name: ENVIRONMENT
          value: production
```

Ajoutez-le avec une fusion stratégique :
```yaml
patchesStrategicMerge:
  - env.yaml
```

---

### Génération Dynamique

Kustomize peut également générer dynamiquement des ConfigMaps et Secrets :

**Génération de ConfigMap**
```yaml
configMapGenerator:
  - name: dynamic-config
    files:
      - configs/config.json
```

**Mise à jour d’images**
```yaml
images:
  - name: ubuntu
    newName: ubuntu
    newTag: 20.04
```

---

### Conclusion

Kustomize est un outil puissant pour gérer les configurations Kubernetes, surtout dans des projets multi-environnements. Contrairement à Helm, il se concentre sur la gestion des configurations existantes sans introduire de dépendances supplémentaires. Sa simplicité et son intégration directe avec `kubectl` en font un outil incontournable pour tout développeur ou administrateur Kubernetes.
