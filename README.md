## Componente de Vue (actualizado a versión 3) para mostrar tabla con paginación y buscador de resultados 

### Instalación
Para poder cargar iconos de la capa de carga y botón de buscar, se usa bootstrap-icons.css:

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.3.0/font/bootstrap-icons.css">
```

el componente debe estar en la carpeta de componentes **resources\js\components** e incluido en el fichero **www\resources\js\app.js** de la siguiente forma:

```js
import './bootstrap';

import {createApp} from 'vue';

import App from './App.vue';
import TomyTable from './TomyTable.vue';

createApp({})
  .component('App', App)
  .component('tomy-table', TomyTable)
  .mount('#app')
```

y ya se podría generar el app.js que será incluido en las páginas de Laravel:
```sh
npm run build
```

para incluirlo en cualquier página se haría mediante la siguiente etiqueta:

```html

    <tomy-table :search="'{{Route('getUsers')}}'"
                :fields="{'name':'Nombre', 'email':'Email', 'active':'Activo'}"
                :buttons="{'bi-bookmark':'{{Route('users.show', 'ID')}}', 'bi-eye':'{{Route('users.edit')}}', 'bi-trash3':'{{Route('users.delete', 'ID')}}'}"
                :extras="{'View':'{{Route('users.show', 'ID')}}', 'Edit':'{{Route('users.edit')}}', 'Delete':'{{Route('users.delete', 'ID')}}'}"
                inputbox
                :tablebuttons="{'Añadir Usuario':'{{Route('users.edit')}}'}"
                :translates="{'search':'Search'}"
                >
    </tomy-table>


```
Donde podemos especificar los siguientes parámetros:
  - **search**: ruta donde se hará la llamada para obtener los resultados 
  - **fields**: los literales a mostrar como cabecera de la tabla (las claves serían los nombres de los campos que devuelve la búsqueda)
  - **buttons**: botones con iconos a mostrar por cada registro
  - **extras**: botones a mostrar en desplegable por cada registro
  - **inputbox**: si aparece este parametro se añadira caja para buscar registros
  - **tablebuttons**: botones a mostrar en la parte de arriba de la tabla

   
### Ejemplo paginación  
Este es un ejemplo de como se cargan los resultados en la **url** que se pasa como parámetro:

```php
 public function search(Request $request){
    if($request->ajax()){
        $search = $request->term;
        return User::where('name', 'like', "%{$search}%")->paginate($request->page_size);        
    }    
}
```