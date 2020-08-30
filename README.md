## Componente de Vue para mostrar tabla con paginación y buscador de resultados

### Instalación
Para poder cargar iconos de la capa de carga y botón de buscar, será necesario instalar la librería **font awesome**:

```html
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css">
```

el componente debe estar en la carpeta de componentes **resources\js\components** e incluido en el fichero **www\resources\js\app.js** de la siguiente forma:

```js
Vue.component('tomy-table', require('./components/TomyTable.vue').default);
```

y ya se podría generar el app.js que será incluido en las páginas de Laravel:
```sh
npm run prod
```

para incluirlo en cualquier página se haría mediante la siguiente etiqueta:

```html
<tomy-table label="cabecera de la tabla" 
            url="{{route('search')}}" 
            :records="15" 
            detail="{{route('detail', 'ID')}}">
</tomy-table>
```
Donde podemos especificar los siguientes parámetros:
  - **label**: el literal que se usará como cabecera de la tabla
  - **url**: ruta donde se hará la llamada para obtener los resultados 
  - **records**: número de registros que se mostrarán por página, por defecto mostrará 10 en caso de no incluirse
  - **detail**: ruta de la pagina de detalle en caso de que la tenga, se sustituirá el literal ID por el campo **id** de los resultados de la paginación. En caso de no incluirse el parametro, se obviará la columna en la tabla
   
### Ejemplo paginación  
Este es un ejemplo de como se cargan los resultados en la **url** que se pasa como parámetro:

```php
 public function search(Request $request){
    if($request->ajax()){
        $users = DB::table('users')
            ->join('roles', 'roles.id', '=', 'users.role_id')
            ->select('users.id', 'users.name as Nombre', 'users.email as Email', 'roles.name as Rol');
        if($request->input('search')){
            $users->where('users.name', 'like', '%' . $request->input('search') . '%')
                ->orWhere('users.email', 'like', '%' . $request->input('search') . '%')
                ->orWhere('roles.name', 'like', '%' . $request->input('search') . '%');
        }
        return response()->json($users->paginate($request->input('records')));
    }    
}
```