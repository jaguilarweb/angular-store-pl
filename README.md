<p align="center">
  <a href="http://angular/" target="blank"><img src="https://angular.io/assets/images/logos/angular/angular.svg" width="200" alt="Angular Logo" /></a>
</p>

# Curso de Consumo de APIs REST con Angular

## Descripción

Este proyecto se deasarrolla en el marco del Curso de Consumo de APIs REST con Angular de Platzi, que usa como base el proyecto creado en el Curso de Angular: Componentes y Servicios.

El proyecto corresponde al Frontend de una tienda digital usando Angular.

## Herramientas

- Node
- Angular
- [Api externa](https://young-sands-07814.herokuapp.com/api/products)
- [Api externa Actualizada](https://fakeapi.platzi.com/en/rest/users)
- [imagenes aleatorias](https://placeimg.com/640/480/any)
- Postman

## Solicitudes HTTP

Para comenzar a trabajar con las solicitudes Http en Angular, necesitaremos verificar que tengamos importado el módulo HttpClientModule de Angular.

### GET

- Obtener todos los productos
- Obtener un producto

En products service tenemos creada una llamada a la api, para obtener los productos de una api anterior, por tanto, modificamos la url para utilizar la nueva api.

Por otra parte, tenemos que hacer las adaptaciones necesarias para la forma en que envía la información la nueva api. Por tanto, ahora debemos modificar nuestro tipado en Models.

Uno de los cambios que vemos es el caso de categoy, que inicialmente la reciviamos como string. Ahora debe recibir un objeto de tipo category.
Una estrategia para enfrentar este cambio, es crear una nueva interfaz con el tipado de cotegory (en el mismo archivo para este ejemplo).


Para obtener el detalle de un producto incorporamos elementos a la vista de product que será un menú lateral retráctil.

Vamos a utilizar la librería swiper y para efectos de seguir el curso incluiremos la version 8, ya que la versión actualizada ya no tiene soporte para componentes de Angular.

Para instalar la versión 8 de swiper, ejecutamos el siguiente comando:

```bash
npm install swiper@8.4
```

En estricto rigor podriamos utlizar cualquier otra librería, por ejemplo:

- Bootstrap (versión ngbootstrap componentes)

### POST

Para crear productos, partimos creando el método createProduct en nuestro respectivo servicio.
Y luego creamos en el componente el endpoint, en este caso los estamos creamdo en products.component.


### DTO: Data Transfer Object

No siempre cohincide con los datos de nuestro modelo de la base de datos.
Por tanto, en lugar de utilizar el DTO en el servicio, utilizaremos un DTO en el componente.

Ejemplo: Cuando modelamos la interfaz para tipar nuestro objeto, en este caso product, incorporamos como atributo la id. No obstante, durante el desarrollo cuando queremos elaborar métodos y necesitamos enviar la data tipada, al emplear la interfaz del modelo para tipar nos exigirá que instanciemos el nuevo objeto incorporando todos los atributos señalados en la interfaz del modelo incluida en este caso la id. El problema es que regularmente la id no la asignaremos manualmente, sino que será asignada por la base de datos cuando se cree el objeto y se guarde en ella. Por tanto, antes de que se guarde el objeto y mientras desarrollamos no tenemos la id del nuevo objeto, que paradojicamente solicita esa id para crearse.
Para resolver esta disyuntiva es que nos sirven los DTO, ya que nos permite tipar el objeto pero con una deseada flexibilidad para incorporar o no atributos que están reflejados en el modelo. De este modo, al crear el DTO nos permite tipar el objeto sin que este me exija la id pero me sigue ayudando con el tipado de los otros atributos.

En este caso, el DTO lo creamo en el mismo archivo del modelo.
Así tenemos en primera instancia la interfaz del producto, y luego creamos una interfaz (dto) especial para el el objeto con el cual se creará un producto propiamente tal. Así en el ejemplo que sigue, luego del Producto tenemos la interfaz CreateProductDTO:

```typescript
export interface Product {
  id: string;
  title: string;
  price: number;
  images: string[];
  description: string;
  category: Category;
}

export interface CreateProductDTO {
  title: string;
  price: number;
  images: string[];
  description: string;
  categoryId: number;
}
```

Ahora, con la finalidad de evitar repetir tantos atributos similares entre las interfaces solemos utilizar una técnica que es extender el comportamiento de una clase a otra y utilizar el Omit, señalando los atributos que omitiremos, y tambiñen podemos señalar un atributo nuevo que requeriremos. Por ejemplo, en este caso para crear solo necesitamos la id de la categoría. No obstante, en el modelo usabamos un objeto categoría.

```typescript
export interface CreateProductDTO extends Omit<Product, 'id' | 'category'> {
  categoryId: number;
}
```

Luego seguimos la misma lógica con todas las peticiones http, creando los métodos en el servicio para recibir dichas peticiones:

- createProduct
- updateProduct
- deleteProduct
- getAllProducts
- getProductById

### Paginación (Parámetros URL)

Para implementar paginación, utilizaremos la técnica del limit y offset.

- limit: cantidad de elementos que queremos obtener
- offset: cantidad de elementos que queremos saltar

Para ello, en el servicio creamos un método que reciba estos parámetros y los envíe a la api.

```typescript
    getProductByPage(limit: number, offset: number){
    return this.http.get<Product[]>(`${this.apiUrl}`, {
      params: {limit, offset}
    });
  }
```

## Observables vs Promesas

### Observables:

- Omitir / responder varias respuestas
- Stream de datos, transmisión de muchos datos
- Permite hacer transformaciones de datos y cancelar la subscripción
- Permite transformar la respuesta (pipes)
- Es posible escuchar constantemente : eventos / responsive / fetchs


### Promesa :

- Retorna una única respuesta
- Solo se ejecuta una vez
- Simplicidad


#### Reintentar peticiones

Esto puede generarse por apuntar mal hacia una dirección url.

Para ello, podemos utilizar el operador retry de rxjs.

```typescript
    getProductByPage(limit: number, offset: number){
    return this.http.get<Product[]>(`${this.apiUrl}`, {
      params: {limit, offset}
    }).pipe(retry(3)); //veces que intentaré la petición
  }
```

También podemos utiliar 'retryWhen' que reintentará la petición cuando se cumpla una condición que podemos especificar.

## Buenas Prácticas

### CORS

Cross-Origin Resource Sharing, es permitir que podamos hacer peticiones desde varios dominios (dominios cruzados).

Este problema tiene que ver con el origen de la petición.

El backend solo acepta peticiones desde el mismo origen, cuando viene de otro dominio muestra un error de Cors.

Habilitar los Cors es que el backend acepte peticiones desde otros dominios, pero también tiene que crear reglas de seguridad.

Ejemplo:
* Todos los dominios

[mydomain, app.mydomain] Acepta dominios especificos

[..., localhost:4200] Acepta desde dominios de desarrollo


A veces no tendremos acceso al backend para que se hagan estas modificaciones, para estos casos Angular nos permite crear un proxy que modifica el origen para igualarlo al del backend (como lo hacen servicios como postman e insomnia, razón por las cuales no generan error de Cors).

No obstante, esta no es una solución definitiva, ya que lo recomendado es que se establezcan las reglas de seguridad en el backend. Además hay que considerar que la solución que otorga Angula es solo para el ambiente de desarrollo. Por tanto, si la aplicación es llevada a producción sin corregir el tema de Cors no será posible corregirlo con el Proxy de Angular.

Para crear el proxy de Angular, haremos lo siguiente:

Crearemos un archivo en el directorio raiz (junto al package.json) llamado proxy.config.json
En el realizamos la configuración:

```json
{
  "/api/*": {
    "target": "https://young-sands-07814.herokuapp.com",
    "secret": true,
    "loglevel": "debug",
    "changeOrigin": true
  }
}
```

Y luego en el package.json escribimos el script:

```json
"start:proxy": "ng serve --proxy-config proxy.config.json",
```

También podemos confirgurar los ambientes para que la url a la api sea escogida dinámicamente, conforme a los archivos de configuración de entorno.

Para ello, en el archivo environment.ts escribimos la url de la api:

```typescript
export const environment = {
  production: false,
  apiUrl: ''
};
```

Y en el archivo environment.prod.ts escribimos la url de la api:

```typescript
export const environment = {
  production: true,
  apiUrl: 'https://young-sands-07814.herokuapp.com'
};
```

Luego, para que el servicio que consume esta api llene automaticamente la url hacemos lo siguiente:

```typescript
import { environment } from './../../environments/environment';

@Injectable({
  providedIn: 'root'
})

export class ProductsService {
  private apiUrl = `${environment.API_URL}/api/products`;
}
```


### Manejo de errores

El manejo de errores puede realizarse de la siguiente forma:


Mensajes de errores provenientes del backend

```ts
  onShowDetail(id: string) {
    this.statusDetail = 'loading';
    this.toggleProductDetail();
    this.productsService.getProductById(id).subscribe((data) => {
      this.productChosen = data;
      this.statusDetail = 'success';
    }, response => {
      //Esta estructura me permite visualizar el manejo de errores que venga del backend.
      console.log(response.error.message);
      this.statusDetail = 'error';
    });
  }
  ```

Manejo de errores desde el servicio


```ts
  getProductById(id: string) {
    return this.http
      .get<Product>(`${this.apiUrl}/${id}`)
      .pipe(catchError((error: HttpErrorResponse) => {
        if(error.status === HttpStatusCode.Conflict) {
          //return throwError('Algo está fallando en el componente'); //Usado en el curso pero deprecado
          return throwError(() => new Error('Algo está fallando en el componente'));
        }
        if(error.status === HttpStatusCode.NotFound) {
          return throwError(() => new Error('El producto no existe'));
        }
        if(error.status === HttpStatusCode.Unauthorized) {
          return throwError(() => new Error('No estás autorizado'));
        }
        return throwError(() => new Error('Algo salió mal'));
      }));
  }
  ```

En lo anterior, podemos observar que Angular nos da diferentes herramientas para manejar los errores.
Por ejemplo:

- throwError : Nos permite personalizar el mensaje del error.
- HttpErrorResponse : Nos permite acceder a la información del error que viene del backend, como por ejemplo el status. Y se utiliza para tipar el error.
- HttpStatusCode : Nos proporciona ayuda con los códigos de los estados.

Una vez que el servicio es quien maneja el error, podemos recibirlo en el componente para que lo envíe a la vista.

```ts
    }, errorMsg => {
      //Esta estructura me permite visualizar el manejo de errores que venga del backend.
      //console.log(response.error.message);
      window.alert(errorMsg)
      this.statusDetail = 'error';
    });
  ```

### Transformar peticiones

En este caso vamos a incorporar un atributo nuevo al modelo de productos, el atributo 'taxes' el cual va a ser opcional.
El valor de este atributo no será proporcionado por el backend sino que será calculado por el frontend.

Esta incorporación la haremos desde el servicio:

```ts
  getAllProducts(limit?: number, offset?: number) {
    let params = new HttpParams();
    if (limit && offset) {
      params = params.set('limit', limit);
      params = params.set('offset', offset);
    }
    return this.http.get<Product[]>(this.apiUrl, { params }).pipe(retry(3), map(products => products.map(item => {
      return {
        ...item,
        taxes: .19 * item.price
      }
    })));
```

Luego debemos asegurarnos de que no se genere un conflicto en la vista, ya que al tener el atributo un valor optativo podría asumir el valor null o undefined y por tanto no podemos imprimir directamente el valor como lo hicimos con los otros atributos:

```html
<p>{{ product.taxes }}</p>
```

Tenemos diferentes maneras de enfrentar esta situación:

- Utilizar el operador Elvis ( ? ) para que imprima el valor solo si existe.

```html
<p>{{ product.taxes ? product.taxes : 'No tiene impuestos' }}</p>
```
- Utilizar el operador (?) como lo muestra el curso. El problema con este es que la consola muestra un warning que puede extenderse al navegador una vez en producción
  
  ```html
  <p>{{ product?.taxes}}</p>
  ```

- Utilizar el operador ngIf para que imprima el valor solo si existe.
  
  ```html
  <p *ngIf="product.taxes">{{ product.taxes }}</p>
  ```

- Utilizar el operador ngIf para que imprima el valor solo si existe, pero en lugar de imprimir el valor, imprimir un mensaje.
  
  ```html
  <p *ngIf="product.taxes; else noTaxes"> {{ product.taxes }}</p>
  <ng-template #noTaxes>
    <p>No tiene impuestos</p>
  </ng-template>
  ```

- Utilizar doble signo de interrogación (??) para que imprima el valor solo si existe, pero en lugar de imprimir el valor, imprimir un mensaje o un valor vacio.
  
  ```html
  <p>{{ product.taxes ?? 'No tiene impuestos' }}</p>
  o
  <p>{{ product.taxes ?? '' }}</p>
  ```

Importante fijarse que metodo se está llamando en el ngInit del componente.
También importante es incorporar el return cada vez que se abre un scope del método 'map' para los array.

### Avoid callback hell
Cuando se trabaja con callbacks, se puede llegar a tener un código muy anidado, lo que se conoce como callback hell.

Ejemplo:

```ts
  readAndUpdate(id: string) {
    this.productsService.getProductById(id)
    .subscribe(data => {
      const product = data;
      this.productsService.updateProduct(product.id, { title: 'change' })
      .subscribe(rtaUpdated => {
        console.log(rtaUpdated);
      })
    })
  }
  ```

En el codigo anterior tenemos un subscribe anidado, y esto podría continuar concatenando subscribes.

Cuando utilizamos promesas las manejamos con los then, ejemplo:

```ts
  readAndUpdate(id: string) {
    this.productsService.getProductById(id)
    .then(data => {
      const product = data;
      this.productsService.updateProduct(product.id, { title: 'change' })
      .then(rtaUpdated => {
        console.log(rtaUpdated);
      })
    })
  }
  ```

No obstante, con los observadores en el caso de tener que recibir un request tras otro podemos utilizar la funcionalidad switchMap de la librería 'rxjs':

```ts
  readAndUpdate(id: string) {
    this.productsService.getProductById(id)
    .pipe(
      switchMap((product) => this.productsService.updateProduct(product.id, { title: 'primer request dependiendo del siguiente' })),
      switchMap((product) => this.productsService.updateProduct(product.id, { title: 'segundo, dependiendo del siguiente' })),
      switchMap((product) => this.productsService.updateProduct(product.id, { title: 'tercero, del que depende el anterior' })),
    )
    .subscribe(data => {
      console.log(data);
    });
  }
  ```

Cuando requiero ejecutar mas de una petición, que no son dependendientes una de otra, pero que necesito ejecutarlas todas juntas trngo las siguientes alternativas:

Si utilizo promesas puedo utilizar el Promise.all:

```ts
  readAndUpdate(id: string) {
    Promise.all([
      this.productsService.getProductById(id),
      this.productsService.getProductById(id),
      this.productsService.getProductById(id),
    ])
    .then(data => {
      console.log(data);
    })
  }
  ```

Si utilizo observables puedo utilizar el zip de 'rxjs':
  
  ```ts
  readAndUpdate(id: string) {
    this.productsService.getProductById(id)
    .pipe(
      switchMap((product) => this.productsService.updateProduct(product.id, { title: 'change' })),
    )
    .subscribe(data => {
      console.log(data);
    });
    zip(
      this.productsService.getProductById(id),
      this.productsService.updateProduct(id, {title: 'nuevo'})
    )
    .subscribe(response => {
      const product = response[0]; //Response 0 sería la primera petición realizada (getProductById) definido por la posición en que se hizo
      const update = response[1]; //Response 1 sería la primera petición realizada (upadteProduct) definido por la posición en que se hizo
    })
  }
  ```


Si utilizo observables puedo utilizar el forkJoin:
  
  ```ts
    readAndUpdate(id: string) {
      forkJoin([
        this.productsService.getProductById(id),
        this.productsService.getProductById(id),
        this.productsService.getProductById(id),
      ])
      .subscribe(data => {
        console.log(data);
      })
    }
  ```

Si bien hemos elaborado toda esta lógica en el componente, lo recomendado es hacerla en el servicio para que se pueda reutilizar el código.

Por tanto, en el servicio creamos la función fetchReadUpdated usando zip:
  
  ```ts
  fetchReadAndUpdate(id: string, dto:UpdateProductDTO){
    return zip(
      this.getProductById(id),
      this.updateProduct(id, dto)
    )
    .subscribe( response => {
      const read = response[0];
      const update = response[1];
    })
  }
  ```

Y luego el componente realiza el siguiente ajuste:

```ts
  readAndUpdate(id: string) {
    this.productsService.fetchReadAndUpdate(id, {title: 'nuevo'});
  }
  ```

## Manejo de autenticación con Jwt

Para lo anterior, exploramos nuestra api y el backend nos debe proporcionar un endpoint donde poder loguear un usuario.
Al realizar el login, la api nos retornará un access token.


### Crear un servicio de autenticación
Creamos un servicio de autenticación que nos permita realizar el login y obtener el access token.

```ts
export class AuthService {
  private apiUrl = `${environment.API_URL}/api/auth`;

  constructor(private http: HttpClient) {}

  login(email: string, password: string) {
    return this.http.post<Auth>(`${this.apiUrl}/login`, {email, password});
  }

  profile(email: string, password: string) {
    return this.http.get(`${this.apiUrl}/profile`);
  }
}
```

Luego desde el controlador invocaremos al servicio de autenticación para acceder al metodo de login, que nos retornará un access token.

```ts
  login() {
    this.authService.login('sebas@gmail.com', '1222')
    .subscribe(rta => {
      console.log(rta.access_token);
    })
  }
```

 y lo haremos accesible mediante un botón desde la vista

```html
<button (click)="login()">Login</button>
```

### Headers

Para enviar el token tenemos diferentes formas de enviarlo mediante los headers:

- Enviar el token en el header de la petición

```ts
  profile(token: string) {
    return this.http.get<User>(`${this.apiUrl}/profile`, {
      headers: {
        Authorization: `Bearer ${token}`,
        // 'Content-type': 'application/json' //optativo
      },
  ```

- Otra forma es utilizando HttpHeaders, para hacerlo de forma dinámica. Especialmente útil si se requiere incorporar una condición.
  
  ```ts
    profile(token: string) {
      const headers = new HttpHeaders({
        Authorization: `Bearer ${token}`,
        // 'Content-type': 'application/json' //optativo
      });
      return this.http.get<User>(`${this.apiUrl}/profile`, { headers });
    }
  ```
  ó
  ```ts
    profile(token: string) {
    const headers = new HttpHeaders();
    headers.set('Authorization', `Bearer ${token}`);
    return this.http.get<User>(`${this.apiUrl}/profile`, {
      headers
    });
  ```

Respuestas de la IA:
- Enviar el token en el body de la petición
  
  ```ts
    profile(token: string) {
      return this.http.get<User>(`${this.apiUrl}/profile`, {
        body: {
          token
        },
        ```


Para enviar el token en el header de la petición, debemos modificar el servicio de autenticación:

```ts
  login(email: string, password: string) {
    return this.http.post<Auth>(`${this.apiUrl}/login`, {email, password})
    .pipe(tap((data: Auth) => {
      localStorage.setItem('token', data.access_token);
    }));
  }
```

Luego desde el componente:

```ts
  getProfile(){
    this.authService.profile(this.token)
    .subscribe((profile) => {
      console.log(profile);
    })
  }
  ```

### Interceptor

Para no tener que estar enviando el token en cada petición, podemos utilizar un interceptor que se encargue de enviar el token en cada petición.

Los interceptores, van a estar en medio de cada solicitud, interceptando y agregando alguna funcionalidad.

Para crear un interceptor, ejecutamos el siguiente comando:

```bash
ng g interceptor interceptors/time --flat
```

Esto nos creará un archivo en el directorio interceptors llamado time.interceptor.ts

Para incorporarlo a la aplicación debemos agregarlo manualmente en el app.module.ts dado que tiene una configuración diferente a otros injectables:

```ts
  providers: [
    {
      provide: HTTP_INTERCEPTORS,
      useClass: TimeInterceptor,
      multi: true
    }
  ],
```

Finalmente, para el ejemplo anterior, veremos en consola que todas las peticiones que realicemos incorporarán el tiempo que han demorado.

Ahora esta funcionalidad la incorporaremos para el manejo de los tokens, ya que en el ambiente de producción lo usual es que todas las peticiones viajen con un token agregado a los headers.

**Nota**:
Durante este apartado veremos 2 interceptores.
- El interceptor que llamaremos time, se debe ejecutar después de la petición ya que mide el tiempo una vez que esta ya se ha ejecutado.
- El interceptor que llamaremos token, se debe ejecutar antes de la petición ya que debe agregar el token antes de que la petición se ejecute.

#### Almacenamiento de tokens

Tipos de almacenamiento:
- In memory storage
- Local Storage
- Session Storage
- Cookies


##### In Memory
Hasta este punto, hemos guardado el token en memoria.

Ejemplo, en el componente nav:

```ts
export class NavComponent implements OnInit {
  token = '';
  profile: User | null = null;
}
```

Esto significa que si el usuario se llega a salir de la aplicación o recarga, ya perdería el token y ya no estaría logueado.

##### LocalStorage o SessionStorage
Estos storage son los que trae el navegador por defecto.
Entre ambos, mejor opción es localstorage ya que mantendrá el valor del token hasta que el usuario cirre sesón, a diferencia del SessionStorage que perderá el token si cerramos el tab y el navegador.

##### Cookie storage

Esta es otra capa, que tiene persistencia y tiene mas seguridad que el LocalStorage.

Para implementar el manejo del token implementaremos el token service, y podremos guardar de diferentes formas el token:

En localStorage:

```ts
  saveToken(token: string) {
    localStorage.setItem('token', token);
    }
  ```

En SessionStorage:
  
  ```ts
    saveToken(token: string) {
      sessionStorage.setItem('token', token);
      }
  ```

En Cookies:
  
  ```ts
    saveToken(token: string) {
      document.cookie = `token=${token}`;
    }
  ```

Para obtener el token desde el local storage

```ts
  getToken() {
    return localStorage.getItem('token');
  }
  ```

Para obtener el token desde el session storage
  ```ts
  getToken() {
    return sessionStorage.getItem('token');
  }
  ```

Para obtener el token desde el cookie storage
  ```ts
  getToken() {
    return document.cookie;
  }
  ```

Para eliminar el token desde el local storage

  ```ts
    removeToken() {
      localStorage.removeItem('token');
    }
  ```

Para eliminar el token desde el session storage

  ```ts
    removeToken() {
      sessionStorage.removeItem('token');
    }
  ```

Para eliminar el token desde el cookie storage
  
  ```ts
    removeToken() {
      document.cookie = 'token=';
    }
  ```

Ahora, vamos a guardar el token de forma automática cuando hagamos el login. Es decir, no vamos a esperar recibirlo en nuestros componentes.

Para lo anterior creamos un token interceptor:

```bash
ng g interceptor interceptors/token --flat
```

Y lo configuramos en el app.module.ts:

```ts
  providers: [
    {
      provide: HTTP_INTERCEPTORS,
      useClass: TokenInterceptor,
      multi: true
    }
  ],
```

Ahora el interceptor agregará el token a las peticiones cuando este (el token), exista. 
Para lo anterior clonará la petición.
Si no exite el token, dejará pasar la petición original.

```ts
  intercept(request: HttpRequest<unknown>, next: HttpHandler): Observable<HttpEvent<unknown>> {
    request = this.addToken(request);
    return next.handle(request);
  }

  private addToken(request: HttpRequest<unknown>){
    const token = this.tokenService.getToken();
    if(token){
      const authReq = request.clone({
        headers: request.headers.set('Authorization', `Bearer ${token}`)
      });
      return authReq;
    }
    return request;
  }
  ```

Y ahora implementamos la nueva lógica en el servicio de auth y en el nav component, pues antes el token se agregaba manual.

En el servicio de token:

```ts
  saveToken(token: string) {
    console.log('Save' + token);
    localStorage.setItem('token', token);
    }

  getToken() {
    return localStorage.getItem('token');
  }
  ```

En el nav component, eliminamos todos los metodos anteriores.
  
  ```ts
  login() {
    this.authService.loginAndGet('sebas@mail.com', '1212')
    .subscribe(user => {
      this.profile = user;
    });
  }
  ```

#### Contexto

El contexto es la manera en que Angular va a saber si tiene que ejecutar o no un interceptor.

Para lo anterior vamos a importar las siguientes dependencias:

```ts
import { HttpContext, HttpContextToken } from '@angular/common/http';

```

Luego en el interceptor creamos una variable global y una función para encender o apagar el contexto:

```ts
const CHECK_TIME = new HttpContextToken<boolean>(() => false);

export function checkTime(){
  return new HttpContext().set(CHECK_TIME, true);
}
```

Incluimos la lógica para evaluar si este contexto se encuentra encendido o apagado:

```ts
intercept(request: HttpRequest<unknown>, next: HttpHandler): Observable<HttpEvent<unknown>> {
    if(request.context.get(CHECK_TIME)) {
      const start = performance.now();
      return next
      .handle(request)
      .pipe(
        tap(() => {
          const time = (performance.now() - start) + 'ms';
          console.log(request.url, time);
        })
      );
    }
    return next.handle(request);
  }
  ```

Y luego debemos incluir en el servicio, en este caso lo haremos en el servicio de product, la acción de encender o apagar dicho contexto:

```ts
getAllProducts(limit?: number, offset?: number) {
    let params = new HttpParams();
    if (limit && offset) {
      params = params.set('limit', limit);
      params = params.set('offset', offset);
    }
    return this.http.get<Product[]>(this.apiUrl, { params, context: checkTime() }).pipe( //en los parametros enviamos el contexto
      retry(3),
      map((products) => {
        return products.map((item) => {
          return {
            ...item,
            taxes: 0.19 * item.price,
          };
        });
      })
    );
  }
  ```


Recordar que si queremos que el contexto esté encendido o no, por defecto, lo debemos establecer cuando definimos la variable global y la función:

```ts
const CHECK_TIME = new HttpContextToken<boolean>(() => false); // Podria estar en true y estaría encendido por defecto

export function checkTime(){
  return new HttpContext().set(CHECK_TIME, true); //De estar encendido por defecto acá ponemos false para apagarlo
}

```

Al hacerlo de la primera manera, es decir definido inicialmente como false, solo enviaremos el contexto en la funciones definidas en los servicios donde queremos que se incluya el contexto. En este caso la incluimos en getAllProducts, por tanto, solo ese método invocará el contexto y para este caso se traducirá que solo ese método medirá la velocidad de la petición.


### Descarga y Carga de archivos con Http

Comunmente nos encontramos que los formularios suele suelen tener la utilidad de carga de un archivo. Un PDF, una foto, etc. Cuando requieres de este tipo de Inputs, tienes que considerar que este archivo está compuesto por datos binarios que tienes que enviar al servidor.

Guardado de archivos binarios en un servidor
En las API REST, el envío de información se realiza en formato JSON con el header Content-type: application/json. No obstante, el envío de archivos binarios, utiliza el header Content-type: multipart/form-data.


#### Descarga de archivos Http

Para realizar la descarga de archivo, podemos utilizar una funcionalidad de los navegadores en html nativo (no de angular) y que es un atributo que agregamos a al elemento 'a':

```html
<a href="./assets/files/texto.txt" download >Descarga</a>
```

No obstante, hay ocasiones en que no podemos utilizar esta opción.
Por ejemplo, cuando queremos descargar un archivo que se encuentra en un servidor externo, o cuando queremos descargar un archivo que se genera dinámicamente.
En estos casos, podemos utilizar el servicio Http para descargar el archivo.

Vamos a crear un servicio para realizar esta funcionalidad:

```bash
ng g s services/files
```

Adicionalmente, vamos a utilizar una dependencia de tercero llamada file-saver y el tipado:

```bash
npm install file-saver
npm install @types/file-saver --save-dev

```

En el servicio creamos el método para descargar los archivos:

```ts
  getFile(name:string, url: string, type: string) {
    return this.http.get(url, {responseType: 'blob'})
    .pipe(
      tap(content => {
        const blob = new Blob([content], {type: type});
        saveAs(blob, name); //Este método es importado desde la libreria file-saver
      }),
      map(() => true)
    );
  }

  ```

En el componente, para descargar programáticamente el archivo definimos el siguiente método:

```ts
  downloadPdf(){
    this.filesService.getFile('my.pdf', 'https://young-sands-07814.herokuapp.com/api/files/dummy.pdf', 'application/pdf')
/*     this.filesService.getFile('my.pdf', './../assets/files/texto.txt', 'application/txt') */
    .subscribe()
  }
  ```

POdemos descargar desde el propio servidor, como de un servidor externo.
Deberemos especificar el formato del archivo en el 3er argumento.

En la vista, agregamos un botón para descargar el archivo:

```html
<button (click)="downloadPdf()">Descargar PDF</button>
```



#### Carga de archivos Http

En el servicio para archivos creamos el método para cargar los archivos:

```ts
  uploadFile(file: Blob){
    const dto = new FormData();
    dto.append('file', file);
    return this.http.post<File>(`${this.apiUrl}/upload`, dto, {
      //Dependiendo del backend podría ser necesario incluir headears. Ejm:
      /* headers: {
        'Content-type' : 'multipart/form-data'
      } */
    })
  }
  ```

En el componente:

```ts

  onUpload(event: Event){
    const element = event.target as HTMLInputElement;
    const file = element.files?.item(0);
    if(file) {
      this.filesService.uploadFile(file)
      .subscribe( rta => {
        this.imgRta = rta.location
      });
    }
  }
  ```

Y en la vista:

```html
<p>
  <input type="file" (change)="onUpload($event)">
  <img *ngIf="imgRta" [src]="imgRta">
</p>
  ```

