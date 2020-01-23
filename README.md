# Gitlab deploy to Kubernates

Данный репозиторий управляет деплоем в кубер. И задействуется репозиторием с кодом через тригер, настраиваемый в Gitlab.

В конфигурации используется простой докер образ [Docker Kubectl](https://github.com/rieset/docker-kubectl) ([hub](https://hub.docker.com/repository/docker/rieset/kubectl)) для доступа к инструментарию kubectl и curl внутри гитлаб ранера

В данном конфиге ожидаем прихода через триггер переменных:
```yaml
 COMMIT_SHORT_SHA - с информацией о коммите в репозитории с кодом
 DOCKER_IMAGE - образ который деплоим
 DOCKER_TAG - тег образа который деплоим 
``` 


## Сервер по умолчанию

По умолчанию запускается сервер из образа `rieset/echo`, который при старте отдает данные в заголовках запроса и `environment` переменные пришедшие на сервер

## Настраиваем Gitlab проект

В настройках проекта `Settings -> CI/CD -> Variables` задаем переменные окружения

```yaml
  KUBE_TOKEN: "..." # Токен доступа к kubernates
  KUBE_URL: "..." # URL кластера  

  PROJECT_NAME: "..." # Название проекта
  PROJECT_NAMESPACE: "..." # Namespace проекта 
```

## Создать Deploy Token

Необходимо в настройках репозитория с кодом проекта создать [Deploy Token](https://gitlab.com/help/user/project/deploy_tokens/index#deploy-tokens)

Данные токена нужно добавить в переменные окружения для того что бы получить возможность из кубера обращаться к образу в registry
```yaml
  $CI_DEPLOY_USER
  $CI_DEPLOY_PASSWORD
```

## Настравиваем секреты

В файле `secrets.yaml` в блоке `stringData` указываем переменные окружения для доступа к другим ресурсам проекта. Например, доступы к БД.

Любые секреты должны быть созданы в настройках данного репозитория, не рекомендуется передавать секреты через тригеры.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ${PROJECT_NAME}-secrets
  namespace: ${PROJECT_NAMESPACE}
type: Opaque
stringData:
  DB_PASSWORD: ${DB_PASSWORD}
  DB_HOST: ${DB_HOST}
  DB_USER: ${DB_USER}
  DB_NAME: ${DB_NAME}
```

В настройках проекта `Settings -> CI/CD -> Variables` задаем переменные окружения, которые будем добавлять в секреты кластера.

```yaml
  # Пример возможных переменных
  DB_PASSWORD: "..."
  DB_HOST: "..."
  DB_USER: "..."
  DB_NAME: "..."
```

Важно: не указывайте ключи доступов в самих файлах. Переменные окружения специально созданы для обеспечения безопасного хранения ключей и паролей


## Настраиваем конфигурацию сервера

По умочанию, такие параметры как namespace и app name указываются в настройках данного репозитория, что бы репозиторий с кодом не мог создавать сервера в произвольных местах кластера

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  # Define in repository config in gitlab
  name: ${PROJECT_NAME}-configs
  # Define in repository config in gitlab
  namespace: ${PROJECT_NAMESPACE}
data:
  # Define in .env file
  PUBLIC_URL: ${PUBLIC_URL}

  # Define in .env file
  API_URL: ${API_URL}

  # Define in .env file
  BROWSER_API_URL: ${BROWSER_API_URL}

```


## Настройка

Открыть настройки проекта в Gitlab 


