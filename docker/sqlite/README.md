
## Ejecuta tu compose
docker compose run sqlite

## Escribe dentro de sqlite
.open prueba.db

docker run --rm \
  -v $(pwd)/sqlite_data:/mcp \
  mcp/sqlite \
  --db-path /mcp/prueba.db \
  write_query --query "CREATE TABLE usuarios (id INTEGER PRIMARY KEY, nombre TEXT, email TEXT);"
