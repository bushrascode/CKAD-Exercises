## Discover and use resources that extend Kubernetes (CRD, Operators)

* [Custom Resources](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/ "Custom Resources")

**1. Create a CustomResourceDefinition for a Car with the following specifications: <br/> &nbsp;&nbsp;&nbsp;&nbsp; a) Kind: Car <br/> &nbsp;&nbsp;&nbsp;&nbsp; b)	Name: cards.stable.example.com <br/> &nbsp;&nbsp;&nbsp;&nbsp; c)	Scope: Namespaced <br/> &nbsp;&nbsp;&nbsp;&nbsp; d)	Singular name: car <br/> &nbsp;&nbsp;&nbsp;&nbsp; e)	Plural name: cars <br/> &nbsp;&nbsp;&nbsp;&nbsp; f)	Properties: <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; i. color: string <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ii. brand: string <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; iii. model: string <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; iv. year: int <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; v. electric: bool**

<details><summary>Solution</summary>

<p>

car-definition.yaml

```YAML
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: cars.stable.example.com
spec:
  group: stable.example.com
  scope: Namespaced
  names:
    plural: cars
    singular: car
    kind: Car
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              color:
                type: string
              brand:
                type: string
              model:
                type: string
              year:
                type: integer
              electric:
                type: boolean
```

```bash
kubectl apply -f car-definition.yaml
kubectl get crd
```

</p>
</details>
<br/>

**2.	Create a custom Car object with the following properties: <br/> &nbsp;&nbsp;&nbsp;&nbsp; a) Color: black <br/> &nbsp;&nbsp;&nbsp;&nbsp; b) Brand: Tesla <br/> &nbsp;&nbsp;&nbsp;&nbsp; c) Model: 3 <br/> &nbsp;&nbsp;&nbsp;&nbsp; d) Year: 2024 <br/> &nbsp;&nbsp;&nbsp;&nbsp; e) Electric: true**

<details><summary>Solution</summary>

<p>

car.yaml

```YAML
apiVersion: stable.example.com/v1
kind: Car
metadata:
  name: tesla
spec:
  color: black
  brand: Tesla
  model: "3"
  year: 2024
  electric: true
```
```bash
kubectl apply -f car.yaml
```
</p>
</details>
<br/>

**3. List all Car resources in the namespace.**

<details><summary>Solution</summary>

<p>

```bash
kubectl get cars
```

</p>
</details>

<br/>