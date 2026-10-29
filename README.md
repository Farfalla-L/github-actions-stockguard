# github-actions-stockguard
Desarrollo de la Practica de GitHub Actions

Captura de la primera ejecución donde falló al correr pytest en local y ocurre porque Python no sabe que debe buscar módulos en la carpeta raíz del proyecto.

<img width="975" height="456" alt="image" src="https://github.com/user-attachments/assets/7def1fa4-3486-4024-b9d7-6f554cd0bba2" />

Captura de la ejecución con el error corregido.  

<img width="975" height="428" alt="image" src="https://github.com/user-attachments/assets/a82689c5-7b41-441c-8645-0d27a6fe6c4f" />
Captura de la ejecución fallida causada intencionalmente 

<img width="975" height="509" alt="image" src="https://github.com/user-attachments/assets/3789ae44-a3e1-4040-8859-98260b09b2f5" />

<img width="975" height="496" alt="image" src="https://github.com/user-attachments/assets/4743a192-5788-4d44-862f-8bba554758c2" />

Captura de la corrección de la del paso anterior 

<img width="975" height="418" alt="image" src="https://github.com/user-attachments/assets/605e3c65-3c33-4b0d-894a-9d52ab4dff50" />

<img width="975" height="441" alt="image" src="https://github.com/user-attachments/assets/5ba31218-096e-4341-8835-7aeb94a1c1e3" />

Pull Request mostrando los checks de Actions 

<img width="975" height="437" alt="image" src="https://github.com/user-attachments/assets/67b0919a-06c3-4cda-abcb-30a8616242a0" />

Ejecución manual mediante Workflow_dispatch
 
<img width="975" height="427" alt="image" src="https://github.com/user-attachments/assets/52ed59a7-ef4a-4cfc-a60b-e365dc7a06d9" />

Preguntas de análisis para responder después de ejecutar

1. Identifica una ejecución exitosa y una fallida. ¿Qué commit o cambio produjo cada una y qué step determinó el resultado?
Ejecución Fallida: En el commit “ci: agregar validación automática con GitHub Actions” en el step determinante “Ejecutar pruebas” se generó porque el runner de GitHub en Ubuntu ejecutó pytest -v, pero al no tener el directorio raíz en el PYTHONPATH, arrojó ModuleNotFoundError: No module named 'src' 
Ejecución Exitosa: En el commit “fix: usar python -m pytest para resolver rutas en CI” en el step determinante “Ejecutar Pruebas” se generó la ejecución exitosa Al cambiar el comando a python -m pytest -v, el intérprete agregó la raíz al sys.path, permitiendo importar el módulo src.inventario y completando los 5 casos de prueba con salida 5 passed in 0.09s.

2. ¿Por qué ejecutar pytest localmente no reemplaza ejecutar pytest en GitHub Actions?

Porque la máquina local contiene un entorno contaminado con configuraciones previas como versiones específicas de Python, librerías instaladas globalmente en Anaconda o variables de entorno que el desarrollador olvidó documentar.
Un ejemplo real fue que en local podíamos tener dependencias o rutas ya resueltas por el IDE, pero al subirlo a GitHub Actions, el runner de Ubuntu arrancó desde una máquina virtual totalmente limpia y nos arrojó de inmediato el error de importación de src. GitHub Actions garantiza que el código sea verdaderamente portátil y reproducible.

3. ¿Qué riesgo tendría hacer merge a main sin esperar los checks del Pull Request?

El riesgo de integrar código roto o con errores de sintaxis a la rama de producción. Esto bloquearía inmediatamente a los otros dos compañeros de equipo al hacer git pull, obligándolos a detener su desarrollo para resolver un fallo que una verificación automática de 10 segundos en el Pull Request habría prevenido.

4. En tu ejecución fallida, copia solo la línea esencial del error y explica qué te permitió diagnosticar.

Tengo dos ejecuciones fallidas en la primera salió el error “ModuleNotFoundError: No module named 'src’” y esto me demostró que el fallo no era de sintaxis ni de dependencias faltantes en requirements.txt, sino de configuración del entorno en el runner de Ubuntu. Al ejecutar pytest de forma aislada, el directorio raíz del repositorio no formaba parte del PYTHONPATH 
Así mismo en el segundo error la línea que salió “AssertionError: assert 'CRITICO' == 'NORMAL'” y esto me mostró con total claridad el contraste entre el valor devuelto por la función y el valor esperado por la prueba unitaria con un stock actual de 7 y un mínimo de 20, la función clasificar_stock evaluó correctamente el estado 'CRITICO'. La línea me permitió diagnosticar que la función estaba operando bien y que el fallo residía exclusivamente en la expectativa errónea 'NORMAL' que colocamos intencionalmente en la aserción de tests/test_inventario.py.

5. ¿Qué diferencia existe entre un step que usa `uses:` y uno que utiliza `run:`?

- uses: invoca una Action empaquetada y reutilizable provista por GitHub o la comunidad.
- run: ejecuta directamente una instrucción de línea de comandos en la terminal del sistema operativo del runner.

6. ¿Qué información aporta `${{ github.actor }}` en `run-name` y por qué puede ser útil en un proyecto en equipo?

Inyecta dinámicamente el nombre de usuario de GitHub del integrante que realizó el push o abrió el Pull Request.
En nuestro equipo de 3 personas para RapidGo 360, esto es vital porque permite verificar en el historial de Actions la participación individual de cada integrante, demostrando quién introdujo un cambio, quién provocó un fallo y quién envió la corrección.

7. Si tu workflow pasa, pero la aplicación completa falla localmente con Docker, ¿significa que Actions “no sirve”? Explica qué está verificando y qué no.

No significa que no sirva los Actions están verificando condiciones estáticas y lógica unitaria:
Qué si verifica Actions: Valida que la sintaxis de Python sea correcta (compileall), que la lógica de las funciones cumpla los casos de negocio (pytest) y que el archivo docker-compose.yml esté bien estructurado.
Qué no verifica Actions: No levanta los contenedores en vivo, no valida si los puertos de red están colisionando en Windows, ni si Kafka está escuchando en localhost:9092 frente a kafka:19092, ni si Spark tiene permisos para crear carpetas de caché Ivy en /nonexistent esto en el proceso de RapidGo.

8. Propón tres checks para tu variante de RapidGo. Para cada uno explica qué error concreto evitaría.

- python -m compileall producer spark tests: Errores tipográficos, de identificación  o sintaxis rota en los scripts de Python antes de que se mezclen a main.
- python -m pytest tests/test_rules.py: Regresiones en la lógica de las alertas de demanda (por ejemplo, que alguien modifique por error el umbral de sobrecarga y la alerta   no se dispare al cruzar los 50 pedidos/minuto).
- docker compose config --quiet: Errores de sintaxis en el archivo docker-compose.yml (claves mal identificadas, variables de entorno no cerradas o mapeos de puertos   inválidos que impedirían arrancar el pipeline al equipo).

9. ¿Cuál de esos checks debería implementarse primero y por qué?

Debe implementarse primero compileall ya que es el check más rápido y económico computacionalmente, si un archivo de Python tiene un paréntesis sin cerrar o una sangría rota, no tiene sentido perder tiempo instalando dependencias en el runner ni ejecutando pruebas de pytest que van a fallar inevitablemente.

10. ¿Qué prueba sería demasiado costosa o frágil para ejecutar en cada push y cómo la validarías entonces?

Levantar el clúster completo de Docker (Kafka, Spark Master, Spark Worker, Postgres y Airflow), esperar a que coordinen por red y transmitir miles de eventos durante 5 o 10 minutos para ver si los archivos Parquet se escriben en el Data Lake. Ejecutar esto en cada push tardaría más de 10 minutos por commit y consumiría rápidamente los minutos gratuitos de GitHub Actions y se valida con los Pull Requests y pushes frecuentes solo se ejecutan pruebas unitarias livianas sobre las funciones de reglas de negocio en Python puro sin levantar Kafka y con La prueba integral pesada con el clúster completo de Docker se ejecuta localmente en las máquinas del equipo antes de crear los releases semanales  o mediante un workflow manual disparado con workflow_dispatch.





