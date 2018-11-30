```
CI_ENVIRONMENT_SLUG="test"
CI_APPLICATION_REPOSITORY='registry.gitlab.com/tkuah/test_rails/master'
CI_APPLICATION_TAG='816a358c2327a06f954d14d6e519b4eada052485'
secret_name=''
POSTGRES_USER='user'
POSTGRES_PASSWORD='testing-password'
POSTGRES_DB=$CI_ENVIRONMENT_SLUG
DATABASE_URL=postgres://${POSTGRES_USER}:${POSTGRES_PASSWORD}@${CI_ENVIRONMENT_SLUG}-postgres:5432/${POSTGRES_DB}
CI_ENVIRONMENT_URL='http://35.201.2.96.nip.io'
KUBE_NAMESPACE='auto-deploy'
VERSION='1'
name="$CI_ENVIRONMENT_SLUG"


helm upgrade --install \
  --wait \
  --set releaseOverride="$CI_ENVIRONMENT_SLUG" \
  --set image.repository="$CI_APPLICATION_REPOSITORY" \
  --set image.tag="$CI_APPLICATION_TAG" \
  --set image.pullPolicy=IfNotPresent \
  --set image.secrets[0].name="$secret_name" \
  --set application.database_url="$DATABASE_URL" \
  --set service.url="$CI_ENVIRONMENT_URL" \
  --set postgresql.nameOverride="postgres" \
  --set postgresql.postgresUser="$POSTGRES_USER" \
  --set postgresql.postgresPassword="$POSTGRES_PASSWORD" \
  --set postgresql.postgresDatabase="$POSTGRES_DB" \
  --namespace="$KUBE_NAMESPACE" \
  --set dbHooks.migrateCommand="gem install bundler && cd /app && bundle exec rake db:setup" \
  --tiller-namespace=auto-deploy \
  --version="$VERSION" \
  "$name" \
  .
```
