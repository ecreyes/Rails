# README

## Crear una aplicación y montar servidor.
Para crear una aplicación en rails se utiliza el siguiente comando:
```bash
rails new nombreapp
```
Montar el servidor:
```bash
rails s
```

Setear página de inicio ir a `config/routes.rb`:
```ruby
root 'welcome#index'
```
Aquí se esta diciendo que el root lo va a manejar el controlador welcome con su método index

## Controladores
Los controladores se encuentran en `app/controllers`, la forma de crear un controlador es:
```bash
rails g controller NombreControlador Metodo1 Metodo2
```
Ejemplo:
```bash
rails g controller Welcome index
```

## ERB y Assets
El ERB permite agregar código en ruby a las vistas en html.
```ruby
<% "Hola"  %> #no muestra lo que esta dentro
<%= "Hola" %> #si muestra el "Hola"

<%[1,2,3,4].each do |number| %>
    <p>Número: <%=number%> </p>
<%end%>
```
En los assets van las img,css,js etc, esto se encuentra en `app/assets`
Una ventaja que tiene rails es que uno simplemente tiene que agregar el css o js dentro del directorio de assets y automáticamente se va a agregar al HTML, esto es por el `*= require_tree .` que viene dentro del archivo respectivo. 
Entonces si se necesita agregar un css o js solo hay que crearlo y ubicarlo en la carpeta de assets, nada de importar en html.

## Modelos.
Para crear un modelo se utiliza la siguiente sintaxis:
```bash
rails g model NombreModelo Campo1:Atributo1 Campo2:Atributo2
```
`NombreModelo` va a ser el nombre de la tabla, siempre escribirlo en singular y ojala en ingles.
`Campo` es la columna de la tabla, puede estar en español.
`Atributo` es el tipo de dato que utiliza la columna.

Ejemlo:
```bash
rails g model Article title:string body:text visits_count:integer
```
Esto creo un archivo `article.rb` que se encuentra en `App/models/article.rb`
El nombre de la tabla creada va a ser `articles`.

Este modelo aun no se encuentra creado en la base de datos,solo hizo todas las configuraciones previas, para realmente crear la tabla en la base de datos se hace una migracion.

## Bases de datos y migraciones.
Las migraciones son archivos que se encargan de hacer modificaciones a la base de datos, estos se almacenan en `db/migrate`.

Para ejecutar las migraciones se utiliza:
```ruby=
rake db:migrate
```
En caso de equivocarnos usamos:
```ruby=
rake db:rollback
```
para deshacer la ultima migracion.

La base de datos se puede ver en el archivo `schema.erb` en la misma carpeta.

Si se desea ver la base de datos en Navicat, solo hay que importar el archivo `development.sqlite3` que se encuentra en `db`, hacer un test y conectarse.

## Layouts
En la carpeta `App/views/layouts` se encuntra el archivo `aplication.html.erb`, lo interesante de este archivo es la parte que contiene `<%=yield%>` que permite que todas las vistas de los controladores que sean hijos de `ApplicationController` posean el mismo codigo del archivo `aplication.html.erb` y solo se modifique la seccion que tiene `yield`.

Es decir `aplication.html.erb` es una vista general que las demas vistas van a heredar el codigo de esta vista, asi el codigo que este en comun para todas las vistas solo es necesario editarlo en `aplication.html.erb` antes de el `yield`.

Un ejemplo de esto va a ser la barra de navegación:
```html
<nav>
<ul>
    <li>Inicio</li>
    <li>Panel Admin</li>
    <li>Contacto</li>
</ul>
</nav>
<%=yield%>
```
En nuestro caso para usar la sintaxis de rails este codigo significa lo mismo:
```ruby
<a href="/"> Inicio </a>
<%=link_to "Inicio",root_path %>
```
La sintaxis es la siguiente:
```ruby
<%=link_to "Nombre",prefix_path %>
```
para ver los `prefix` se utiliza en la consola:
```bash
rake routes
```
Resultando:
```html
<nav>
<ul>
    <li>
        <%=link_to "Inicio",root_path %>
    </li>
    <li>Panel Admin</li>
    <li>Contacto</li>
</ul>
</nav>
```

## Rutas
Las rutas de nuestro proyecto se encuentran en `config/routes.erb`
Los verbos `HTTP` utilizados en ruby on rails son los siguientes:
* GET "datos por url"
* POST "datos por debajo o formularios"
* PATCH
* PUT
* DELETE "eliminar datos"

En rails cuando se tiene la ruta:
```ruby
get "welcome/index"
```
se refiere al verbo `GET`,luego debe ir al controlador `welcome` a la accion `index`.

Si uno quisiera cambiar la dirección a otra cualquiera se utiliza:
```ruby
get "nombre" to: "welcome#index"
```
donde `nombre` puede ser cualquier nombre y se refiere  al mismo controlador y accion que antes.

Si se crea la ruta:
```ruby
resources :articles
```
Es lo mismo que tener:
```ruby
get "/articles" #INDEX
post "/articles" #CREATE
delete "/articles/:id" #DESTROY
get "/articles/:id" #SHOW
get "/articles/new" #NEW
get "/articles/:id/edit" #EDIT
patch "/articles/:id" #UPDATE
put "/articles/:id" #UPDATE
```
Los comentarios son las acciones que se buscan en el controlador.
* Index muestra todos los articulos.
* Show solo muestra 1 articulo.
* New es el formulario que se envia en caso de crear un articulo pero solo esta en la memoria.
* Create crea el articulo en la base de datos.
* Delete elimina el articulo de la base de datos


Tambien se pueden excluir algunas rutas de la siguiente forma:
```ruby
resources :articles, except: [:delete] 
```
o sino:
```ruby
resources :articles, only: [:create,:show]
```
que solamente crea esas 2 rutas.

## Pasando datos de controlador a vista
Al crear `resources :articles` si uno trata de entrar a cualquier vista, como la `http://localhost:3000/articles` por ejemplo, no se va a poder y va a mandar un error `uninitialized constant ArticlesController`.Para solucionar este error se tiene que crear el controlador con:
```bash
rails g controller Articles
```
Recordar que esto crea un controlador en `app/contollers` con el nombre de `articles_controller` y crea una carpeta en views.

Ahora si se trata de acceder por get `http://localhost:3000/articles` muestra el siguiente error `The action 'index' could not be found for ArticlesController` esto significa que hay que crear la accion index en el controlador:
```ruby
def index
end
```
Ahora si se trata de acceder muestra el siguiente error: `ActionController::UnknownFormat in ArticlesController#index`, esto quiere decir que encontro la accion pero no una vista asociada por lo que hay que crearla en `app/views/articles` con el nombre de `index.html.erb`.

Como ya se tiene el modelo creado en `app/models` en el archivo `article.rb` la primera linea es `class Article < ApplicationRecord`, entonces hay que usar `Article` como active record en `articles_controller`.

resultando en el metodo index:
```ruby
def index
    @articles = Article.all
    puts "Entro a index" #console.log js
end
```
donde `@articles` es una variable de clase, esto permite que la variable pueda ser accedida desde la vista como del controlador.Las variables que empiezan sin el `@` solo son accedidas por el controlador.

En la vista de index para inspeccionar la variable `@articles` se utiliza:
```ruby
<%= @articles.inspect %>
```

Para mostrar los elementos con `show`, se crea la vista `show.html.erb` y en el controlador se crea la accion:
```ruby
def show
    @article = Article.find(params[:id])
    puts params #console.log de js
end
```
`params` es como un diccionario que recibe todos los datos que son enviados, ya sea por formularios, urls, etc.

Finalmente las vistas quedan:

index:
```ruby
<% @articles.each do |t| %>
	<h1><%= t.title %></h1>
	<div><%= t.body %></div>
<%end%>	
```

show:
```ruby
<h1><%= @article.title%></h1>
<div><%= @article.body %></div>
```