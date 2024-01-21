# Helm Chart for LiteLLM

## Example Postgres Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: litellm-dbcredentials
data:
  # Password for the "postgres" user
  postgres-password: <some secure password, base64 encoded>
  password: <some secure password, base64 encoded>
type: Opaque
```