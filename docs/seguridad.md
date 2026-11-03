# Seguridad del gestor

| Comprobación | Antes | Después |
|---|---|---|
| Cabecera Server | Oculta versión (`Server: nginx`) | Oculta versión (`Server: nginx`) |
| /xmlrpc.php | Bloqueado (403 Forbidden) | Bloqueado (403 Forbidden) |
| /wp-config.php | Bloqueado (403 Forbidden) | Bloqueado (403 Forbidden) |
| PHP en uploads | Bloqueado (403 Forbidden) | Bloqueado (403 Forbidden) |
| Edición de ficheros desde el panel | Habilitada | Deshabilitada (`DISALLOW_FILE_EDIT`) |
| Límite de intentos de login | Inexistente (Fuerza bruta posible) | Activo (Limit Login Attempts) |

## Medidas aplicadas y por qué

1. **Ocultación de la firma del servidor (`server_tokens off`):** Impide que atacantes o escáneres automáticos identifiquen la versión exacta de Nginx para explotar vulnerabilidades conocidas (CVEs)[cite: 1].
2. **Bloqueo de acceso a `/xmlrpc.php` y `/wp-config.php`:** Previene ataques de fuerza bruta remotos, amplificación de denegación de servicio (DDoS) y el acceso a la configuración crítica del sistema[cite: 1].
3. **Denegación de ejecución de PHP en `/wp-content/uploads/`:** Bloquea la ejecución remota de código (RCE) en caso de que un atacante consiga subir una webshell o un archivo `.php` malicioso a través de la carpeta de subidas de medios[cite: 1].
4. **Desactivación del editor de archivos (`DISALLOW_FILE_EDIT`):** Deshabilita la edición de temas y plugins desde el panel de administración, evitando que un usuario con credenciales de admin comprometidas pueda inyectar código PHP ejecutable en el servidor[cite: 2].
5. **Separación de roles de usuario y límite de intentos de login:** Aplicación del principio de mínimo privilegio asignando el rol Editor para el trabajo diario y protección contra ataques de fuerza bruta limitando las tentativas en `/wp-login.php`[cite: 2, 3].
