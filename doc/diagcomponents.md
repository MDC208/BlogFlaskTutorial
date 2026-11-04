# Diagrama de Components

Descriure tots els components de l'aplicació

Components: db, _init_, auth, blog

## db
Descripció: funcions d'accés a BBDD
Mètodes:
- get_db(): Retorna connexió, metode que retorna la connexió a BBDD
- close_db(): Tanca la connexió a BBDD
- init_db(): Elimina les dades existents i crea noves taules
- init_db_command(): Fa un crit a init_db
- init_app(): Registra les funcions de la base de dades amb l'aplicació Flask

## blog
Descripció: Funcions de control de la aplicació
Mètodes:
- index(): Mostra totes les publicacions, per ordre de més recents
- get_post(): Agafa una publicació i al autor mitjançant el seu ID
- create(): Crea una nova publicació pel usuari actual
- update(): Actualitza una publicació si l'usuari es l'autor
- delete(): Elimina una clase

## auth
Descripció: Funcions d'inici de sessió
Mètodes:
- login_required(): Porta l'usuari al menú d'inici de sessió
- load_logged_in_user(): Inicia sessió automaticament si ja si ha iniciat abans
- register(): Registra nous usuaris
- login(): inicia sessió en un usuari ja registrat
- logout(): tanca la sessió actual

## _init_
Descripció: Funcions per a inicialitzar la aplicació
Mètodes:
- create_app(): Crea i configura la aplicació flask

```mermaid
classDiagram
  class DB {
    +get_db()
    +close_db()
    +init_db()
    +init_db_command()
    +init_app()
  }
  class Blog {
    +index()
    +get_post()
    +create()
    +update()
    +delete()
  }
  class Auth {
    +login_required()
    +load_logged_in_user()
    +register()
    +login()
    +logout()
  }
  class _init_ {
    +create_app()
  }
  _init_ --> DB
  _init_ --> Blog
  _init_ --> Auth
```

# Diagrames de seqüencia 
## Diagrama init database
```mermaid
sequenceDiagram
    autonumber
    actor U as Usuari (terminal)
    participant CLI as Flask CLI<br/>(FlaskGroup / ScriptInfo)
    participant Init as flaskr/__init__.py<br/>create_app()
    participant App as Flask app
    participant DB as flaskr/db.py
    participant Click as init_db_command<br/>(@with_appcontext)
    participant FS as Sistema de fitxers
    participant SQL as SQLite<br/>(instance/flaskr.sqlite)

    U->>CLI: flask --app flaskr init-db
    CLI->>CLI: Parseja --app flaskr
    CLI->>Init: Importa el paquet flaskr i localitza create_app()
    Init->>App: Flask(__name__, instance_relative_config=True)
    Init->>App: config.from_mapping(SECRET_KEY, DATABASE=instance/flaskr.sqlite)
    Init->>App: config.from_pyfile("config.py", silent=True)
    Init->>FS: os.makedirs(app.instance_path)
    Init->>DB: db.init_app(app)
    DB->>App: teardown_appcontext(close_db)
    DB->>App: cli.add_command(init_db_command)
    Init->>App: register_blueprint(auth.bp), register_blueprint(blog.bp)
    Init->>App: add_url_rule("/", endpoint="index")
    Init-->>CLI: retorna app

    CLI->>Click: Resol la subcomanda "init-db" i la invoca
    CLI->>App: Empeny l'app context (with_appcontext)
    Click->>DB: init_db()
    DB->>DB: get_db()
    DB->>SQL: sqlite3.connect(current_app.config["DATABASE"])
    SQL-->>DB: connexió
    DB->>DB: g.db = connexió, row_factory = sqlite3.Row
    DB->>FS: current_app.open_resource("schema.sql")
    FS-->>DB: contingut de schema.sql
    DB->>SQL: executescript(DROP TABLE IF EXISTS... CREATE TABLE user, post)
    SQL-->>DB: taules creades
    Click->>U: click.echo("Initialized the database.")

    CLI->>App: Surt de l'app context
    App->>DB: teardown_appcontext → close_db()
    DB->>SQL: db.close()
```

## Diagrama de arrencada de la app
```mermaid
sequenceDiagram
    autonumber
    actor U as Usuario (terminal)
    participant CLI as Flask CLI<br/>(run_command)
    participant Init as flaskr/__init__.py<br/>create_app()
    participant App as Flask app
    participant DB as flaskr/db.py
    participant Auth as flaskr/auth.py
    participant Blog as flaskr/blog.py
    participant Srv as Servidor de desarrollo

    U->>CLI: flask --app flaskr run --debug
    CLI->>CLI: Parsea --app flaskr y --debug
    CLI->>Init: Importa el paquete flaskr y localiza create_app()
    Init->>App: Flask(__name__, instance_relative_config=True)
    Init->>App: config.from_mapping(SECRET_KEY, DATABASE)
    Init->>App: config.from_pyfile("config.py", silent=True)
    Init->>App: os.makedirs(app.instance_path)
    Init->>DB: db.init_app(app)
    DB->>App: teardown_appcontext(close_db)
    DB->>App: cli.add_command(init_db_command)
    Init->>Auth: import auth
    Init->>App: register_blueprint(auth.bp)
    Auth->>App: before_app_request(load_logged_in_user)
    Init->>Blog: import blog
    Init->>App: register_blueprint(blog.bp)
    Init->>App: add_url_rule("/", endpoint="index")
    Init-->>CLI: devuelve app
    CLI->>Srv: Inicia el servidor (127.0.0.1:5000)
    Srv-->>U: Running on http://127.0.0.1:5000
```

## Diagrama de la petició get / login
```mermaid
sequenceDiagram
    autonumber
    actor B as Navegador
    participant Srv as Servidor de desarrollo
    participant App as Flask app
    participant Auth as flaskr/auth.py
    participant Blog as flaskr/blog.py
    participant DB as flaskr/db.py
    participant SQL as SQLite<br/>(instance/flaskr.sqlite)

    B->>Srv: GET /
    Srv->>App: Llamada WSGI (app.wsgi_app)
    App->>App: Crea app context y request context
    App->>Auth: before_app_request → load_logged_in_user()
    Auth->>App: Lee session.get("user_id")
    alt No hay user_id en la sesión
        Auth->>App: g.user = None
    else Hay user_id
        Auth->>DB: get_db()
        DB->>SQL: sqlite3.connect(DATABASE)
        SQL-->>DB: conexión
        DB-->>Auth: db (g.db)
        Auth->>SQL: SELECT * FROM user WHERE id = ?
        SQL-->>Auth: usuario
        Auth->>App: g.user = usuario
    end
    App->>Blog: Resuelve "/" → blog.index()
    Blog->>DB: get_db()
    DB-->>Blog: db (g.db)
    Blog->>SQL: SELECT p.id, title, body, created, author_id, username<br/>FROM post p JOIN user u ... ORDER BY created DESC
    SQL-->>Blog: lista de posts
    Blog->>App: render_template("blog/index.html", posts=posts)
    App-->>Blog: HTML renderizado (base.html + index.html)
    Blog-->>App: respuesta
    App->>DB: teardown_appcontext → close_db()
    DB->>SQL: db.close()
    App-->>Srv: Respuesta 200 OK
    Srv-->>B: HTML
```
