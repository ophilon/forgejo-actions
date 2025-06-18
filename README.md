# proba Action ssh-agent

example for .gitlab-ci.yaml:

```
compose:
  stage: compose
  rules:
    - if: $CI_COMMIT_BRANCH
      changes:
        paths:
        - frontend/**/*
      variables:
        BUILD_FRONT: "BUILD_FRONT"
      when: manual
    - if: $CI_COMMIT_BRANCH
      changes:
        paths:
        - backend/**/*
        - backend-report/**/*
        - app/**/*
        - .gitlab-ci.yml
      when: manual
  image: vault:1.11.3
  before_script:
    - apk add openssh-client bash
    - eval $(ssh-agent -s)
    - echo "$(echo $SSH_PRIVATE_KEY|base64 -d)" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - echo "$SSH_KNOWN_HOSTS" >> ~/.ssh/known_hosts
    - chmod 644 ~/.ssh/known_hosts
  script:
    - export VAULT_TOKEN="$(vault write -field=token auth/jwt/login role=sausage-store jwt=$CI_JOB_JWT)";
      export SPRING_DATASOURCE_PASSWORD="$(vault kv get -field=spring.datasource.password secret/sausage-store)";
      export SPRING_DATA_MONGODB_URI="$(vault kv get -field=spring.data.mongodb.uri secret/mongodb.uri)";
      ssh ${DEV_USER}@${DEV_HOST} "export
       'BRANCH=${CI_COMMIT_REF_NAME}'
       'VERSION=${VERSION}'
       'NEXUS_REPO_URL=${NEXUS_REPO_URL}'
       'NEXUS_REPO_USER=${NEXUS_REPO_USER}'
       'NEXUS_REPO_PASS=${NEXUS_REPO_PASS}'
       'NEXUS_MVN2_FOLDER=${NEXUS_MVN2_FOLDER}'
       'MVN_PATH=${MVN_PATH}'
       'GITLAB_REPO=${GITLAB_REPO}'
       'BUILD_FRONT=${BUILD_FRONT}';
        setsid /bin/bash -s " < docker_compose.sh
```
