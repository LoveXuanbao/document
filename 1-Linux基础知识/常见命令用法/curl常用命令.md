### 创建nacos命名空间

```bash
curl -X POST 'http://10.202.43.46:30848/nacos/v1/console/namespaces' -d 'customNamespaceId=dev&namespaceName=dev&namespaceDesc=dev'
```

### 导入nacos配置

```bash
curl --location --request POST 'http://10.202.43.46:30848/nacos/v1/cs/configs?import=true&namespace=yingyongkaifajichengguanlipingtai' --form 'policy=OVERWRITE' --form 'file=@"/data1/lawpass/packages/nacos_config_export_20240527235045.zip"'
```

