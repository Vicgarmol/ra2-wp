# Informe RA3 - Práctica 3: Importar y exportar contenidos en distintos formatos

**Alumno/a:** Víctor  
**Modulo:** Implantación de Aplicaciones Web (2º ASIR)  
**Directorio de trabajo:** `ra2-wp`  

---

## 1. Cuatro formatos, cuatro usos

| Formato | Qué lleva | Para qué sirve / Caso de uso |
| :--- | :--- | :--- |
| **WXR (XML de WordPress)** | Entradas, páginas, categorías, etiquetas, comentarios y autores | Trasladar contenido estructurado entre distintas instancias de WordPress |
| **CSV** | Datos tabulares organizados en filas y columnas con cabecera | Exportar a hojas de cálculo (LibreOffice Calc/Excel) o importar usuarios masivamente |
| **JSON (API REST)** | Objetos de datos estructurados e intercambiables | Permitir la comunicación bidireccional entre aplicaciones externas (móviles, headless CMS) y WordPress |
| **SQL** | Esquema y datos de la base de datos completa o tablas específicas | Realizar copias de seguridad de bajo nivel y migraciones completas del sistema |

---

## 2. Exportar el sitio en WXR

- **Comando ejecutado:**
  ```bash
  mkdir -p exportaciones
  ./wp export --stdout > exportaciones/sitio_$(date +%Y%m%d).xml
  grep -c '<item>' exportaciones/sitio_*.xml
  grep -o '<wp:post_type>.*</wp:post_type>' exportaciones/sitio_*.xml | sort | uniq -c
  ```

- **Resultados obtenidos:**
  - Número de elementos `<item>` exportados: *Indicar la cifra obtenida por grep*
  - Desglose por tipos de contenido (`wp:post_type`): `post`, `page`, `nav_menu_item`, `wp_template`, `wpcf7_contact_form`.

- **Justificación técnica:**  
  El parámetro `--stdout` escribe la salida XML directamente en el flujo estándar para redirigirlo al archivo local con `>`. Se utiliza el script `./wp` porque ejecuta el comando dentro del contenedor manteniendo la codificación adecuada y evitando saltos de línea adicionales que corruptos en la estructura XML.

---

## 3. Importar contenidos de ejemplo (WXR)

- **Comandos ejecutados:**
  ```bash
  ./wp plugin install wordpress-importer --activate
  ./wp import /datos/contenidos_asir.xml --authors=create
  ./wp post list --post_type=post,page --fields=ID,post_type,post_title,post_author
  ./wp term list category --fields=name,count
  ```

- **Verificación:**
  - **Estado de la importación:** `Success: Finished importing from '/datos/contenidos_asir.xml' file.`
  - **Nuevas entradas creadas:** 6 entradas distribuidas entre las categorías *Noticias*, *Eventos* y *Prácticas*.
  - **Nuevas páginas importadas:** «Horario del ciclo» y «Normas del aula técnica».
  - **Gestión de autores:** El modificador `--authors=create` creó automáticamente las cuentas asignadas o las vinculó a los usuarios existentes si el nombre de usuario ya coincidía (ej. `coord.web`).

---

## 4. Importar usuarios desde CSV

- **Comandos ejecutados:**
  ```bash
  head -n 3 datos/usuarios.csv
  ./wp user import-csv /datos/usuarios.csv
  ./wp user list --fields=user_login,display_name,roles
  ```

- **Observaciones de seguridad y ejecución:**
  - **Creación vs. Actualización:** Para los usuarios nuevos se devolvió el estado `Success: Created user`, mientras que para las cuentas préviamente existentes (como `coord.web` o `profe.redes`) se indicó `Success: Updated user`, actualizando sus atributos sin duplicar perfiles.
  - **Gestión de contraseñas:** El archivo CSV no contiene contraseñas en texto plano por motivos de seguridad. WP-CLI genera automáticamente una clave aleatoria segura para cada usuario importado.

---

## 5. Exportar listados en CSV y JSON

- **Comandos ejecutados:**
  ```bash
  ./wp post list --post_type=post --fields=ID,post_title,post_date,post_status --format=csv > exportaciones/entradas.csv
  ./wp user list --fields=user_login,user_email,roles --format=json > exportaciones/usuarios.json
  ```

- **Comprobación de la salida:**
  - `entradas.csv`: Archivo en formato plano separado por comas. Al abrirlo en herramientas como LibreOffice Calc debe seleccionarse la coma `,` como delimitador de campo y codificación `UTF-8` para la correcta representación de caracteres especiales y tildes.
  - `usuarios.json`: Representación estructurada en JSON procesada mediante `python3 -m json.tool`.

---

## 6. JSON desde la API REST

- **Comandos ejecutados:**
  ```bash
  curl -s "http://localhost:8080/wp-json/wp/v2/posts?per_page=3&_fields=id,title,link" | python3 -m json.tool
  curl -s "http://localhost:8080/wp-json/wp/v2/users?_fields=id,name" | python3 -m json.tool
  ```

- **Análisis técnico:**
  - La API REST expone en `/wp-json/` los datos del sitio de forma accesible sin requerir autenticación para lecturas públicas.
  - En caso de recibir un error `404` o `rest_no_route`, es indispensable habilitar la estructura de enlaces permanentes (*permalinks*) desde el panel de administración o consultar pasando la ruta por parámetro query string (`?rest_route=/wp/v2/posts`).

---

## 7. SQL: Exportar una tabla y entender el riesgo

- **Comando ejecutado:**
  ```bash
  docker compose exec -T db sh -c 'mariadb-dump --no-tablespaces -u"$WP_DB_USER" -p"$WP_DB_PASSWORD" "$WP_DB_NAME" wp_users' > exportaciones/wp_users.sql
  grep -o "'\$wp\$[^']*'" exportaciones/wp_users.sql | head -n 3
  ```

- **Evaluación de seguridad:**
  - Las contraseñas de la tabla `wp_users` aparecen cifradas con el algoritmo bcrypt (cadenas que inician con `$wp$2y$`).
  - **Riesgo:** Aunque no pueden descifrarse directamente, si un atacante obtiene el archivo `.sql` puede realizar ataques de fuerza bruta u orquestar ataques por diccionario offline.
  - **Control de versión:** Se comprueba con `git status` que las carpetas `exportaciones/` y `backups/` no figuran para seguimiento, estando protegidas mediante el archivo `.gitignore`.

---

## 8. Intercambio cooperativo

1. **Entrada redactada en la categoría *Prácticas*:**
   > "WXR permite trasladar contenidos y autores entre sitios WordPress. CSV es ideal para manipular tablas de datos e importar listados masivos. JSON facilita la integración de aplicaciones mediante la API REST. SQL guarda la estructura y datos de la base de datos para migraciones completas."

2. **Proceso de exportación e importación cruzada:**
   - Exportación de la entrada filtrando por la categoría **Prácticas** a un archivo XML desde el panel de WordPress (**Herramientas → Exportar**).
   - Transferencia del fichero a la pareja de prácticas e importación en el sitio destino mediante **Herramientas → Importar → WordPress**.
   - **Resultado:** La entrada enviada por la pareja aparece publicada en el sitio local mostrando su correspondiente atribución de autoría.