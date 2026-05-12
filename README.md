# Redis Cluster Deployment

Documentação para implantação e configuração do Redis Cluster no ambiente da Avançada.

---

# Instalação do Redis Cluster

## 1. Remover instalação anterior do Redis Cluster

Execute o comando abaixo para remover a instalação existente do Redis Cluster:

```bash
helm uninstall redis-cluster -n govbr
```

---

## 2. Instalar o Redis Cluster

Realize a instalação utilizando o chart Helm e o arquivo `values.yaml`:

```bash
helm install redis-cluster . -f values.yaml -n govbr
```

---

## 3. Remover PVCs antigos do Redis

Após a remoção do cluster anterior, exclua todos os PVCs relacionados ao Redis para evitar inconsistências de dados.

Exemplo:

```bash
kubectl delete pvc -n govbr -l app=redis-cluster
```

> Ajuste o label conforme a configuração utilizada no ambiente.

---

## 4. Aguardar inicialização dos pods

Aguarde até que todos os containers do Redis estejam em estado `Running`.

Verifique utilizando:

```bash
kubectl get pods -n govbr
```

---

# Criação de Usuário no Redis

## 5. Acessar a CLI do Redis

Entre em um dos pods do Redis, por exemplo o pod `redis-cluster-0`:

```bash
kubectl exec -it redis-cluster-0 -n govbr -- bash
```

Depois acesse a CLI do Redis:

```bash
redis-cli -h 127.0.0.1
```

---

## 6. Criar usuário no Redis

Execute o comando abaixo para criar o usuário:

```bash
ACL SETUSER govbr on >$PASSWORD ~* &* +@all
```

### Observações

- Substitua `$PASSWORD` pela senha definida no `values.yaml`
- Utilize o valor presente na branch mais recente de `staging` ou `prod`
- pesquise por redis no values padrão.

---

## 7. Reiniciar o OAuth

Após a criação do usuário no Redis, reinicie a aplicação OAuth para que ela estabeleça conexão utilizando as novas credenciais.

Exemplo:

```bash
kubectl rollout restart deployment oauth -n govbr
```

---

# Verificações

## Validar conexão com Redis

Você pode validar os usuários criados executando:

```bash
ACL LIST
```

---

