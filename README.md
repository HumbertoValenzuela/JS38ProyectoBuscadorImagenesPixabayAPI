# JS38ProyectoBuscadorImagenesPixabayAPI
JS 38. PROYECTO Buscador de Imagenes con Pixabay API
* https://pixabay.com/api/docs/ ir a la página. Iniciar sesión - get started - docs
* Buscar la Key y el ejemplos de URL

```javascript
// 38. PROYECTO Buscador de Imagenes con Pixabay API
// https://pixabay.com/api/docs/ ir a la página
// iniciar sesión - get started - docs
//  buscar la Key y el ejemplos de URL


// Selectores
const resultado = document.querySelector('#resultado');
const formulario = document.querySelector('#formulario');

// Paginación
const registrosPorPagina = 40;// para mostrar 40 imagenes por página
let totalPaginas;// Cada vez que se realiza una consulta este valor cambia
let iterador;
const paginacionDiv = document.querySelector('#paginacion');
let paginaActual = 1; //Inicio de página es siempre la 1

window.onload = () => {
    formulario.addEventListener('submit', validarFormulario);
}

function validarFormulario(e) {
    e.preventDefault();

    // Selector input para ingresar texto de busqueda
    const terminoBusqueda = document.querySelector('#termino').value;

    // Validar Termino de busqueda
    if (terminoBusqueda === '') {
        mostrarAlerta('Agrega un término de búsqueda');
        return;
    }

    // Buscar Imagen en pixsabit
    buscarImagenes();
}

function mostrarAlerta(mensaje) {

    // Selector que se agrega y se quita con scripting
    const existeAlerta =  document.querySelector('.bg-red-100');

    // Validar si existe la clase bg-red-100 
    if(!existeAlerta) {
        // Scripting
        const alerta = document.createElement('p');
        alerta.classList.add('bg-red-100', 'border-red-400', 'text-red-700', 'px-4', 'py-3', 'rounded', 'max-w-lg','mx-auto', 'mt-6', 'text-center');

        alerta.innerHTML = `
            <strong class="font-bold">Error!</strong>
            <span class="block sm:inline">${mensaje}</span>
            `;

            formulario.appendChild(alerta);

            setTimeout(() => {
                alerta.remove()
            }, 3000);
    }
}

// Utilizando la APi de pixabay
// Quitar parametro.
// para solucionar tema parametro es colocar una var local
function buscarImagenes() {

    const termino = document.querySelector('#termino').value;

    const key = '21291177-f415154980c4ff4d84ba79eb4';
    // Por default trae 20 imagenes, para modificar
    // const url = `https://pixabay.com/api/?key=${key}&q=${terminoBusqueda}&per_page=100`;
    const url = `https://pixabay.com/api/?key=${key}&q=${termino}&per_page=${registrosPorPagina}&page=${paginaActual}`;

    console.log(url);// Probar si funciona la url y la key
    fetch(url)
        .then(respuesta => respuesta.json())
        .then( resultado => {
            console.log(resultado);// ver el objeto y sus atributos

            // Obtener el total de imagenes. totalHits es una propiedad de pixybay. Propiedad total de pixabay no funcionará debido a que se paga $Dinero
            totalPaginas = calcularPaginado(resultado.totalHits);
            console.log(totalPaginas);//Ver resultado de calcularPaginado
            mostrarImagenes(resultado.hits);
        })
}

function calcularPaginado(total) {
    // console.log(total);
    return parseInt(Math.ceil(total / registrosPorPagina));
}

// Generador que va a registrar la cantidad de elementos de acuerdo a las páginas
function *crearPaginador(total) {
    console.log(total);
    for(let i=1; i <= total; i++) {
        console.log(i);
        // para registrar los valores internamente en el generador
        yield i;
    }
}



// ver el resultado de buscar imagenes
function mostrarImagenes(imagenes) {
    limpiarHTML(resultado);
    console.log(imagenes);// entrega una Array
    
    // Iterar sobre el arreglo de imagenes y constriur el HTML
    imagenes.forEach(imagen => {
        //Datos sacados del Array entregado por pixabay
        const { previewURL, likes, views, largeImageURL } = imagen;

        // div tamaño la mita, Mediano tercera parte, largo una cuarta parte
        // target="_blank" rel="noopener noreferrer" para evitar problema de seguridad
        resultado.innerHTML += `
        <div class="w-1/2 md:w-1/3 lg:w-1/4 p-3 mb-4" >
            <div class="bg-white" >
                <img class="w-full" src="${previewURL}" >

                <div class="p-4" >
                    <p class="font-bold"> ${likes} <span class="font-light"> Me Gusta</span></p>
                    <p class="font-bold"> ${views} <span class="font-light"> Veces Vista</span></p>

                    <a class="block w-full bg-blue-800 hover:bg-blue-500 text-white font-bold text-center rounded mt-5 p-1" href="${largeImageURL}" target="_blank" rel="noopener noreferrer">Ver Imagen </a>
            </div>
        </div>    
        `; 
    });
    limpiarHTML(paginacionDiv);
    imprimirPaginador();
    // iterador = crearPaginador(totalPaginas);
    // console.log(iterador.next());
      
}

function imprimirPaginador() {
    
    iterador = crearPaginador(totalPaginas);
   // console.log(iterador);//Para sacarlo del estado Suspended usar .next
    //console.log(iterador.next().value);//{value: undefined, done: true}
    // done:true indica que ya llego al final 
    // console.log(iterador.next().done);

    // while tendrá valor true se ejecutará todo el tiempo
    while (true) {
        const { value, done} = iterador.next();
        if (done){console.log(done); return;} //si ya llegamos al final, dejar de ejecutar done: cambia a true
       
        // Caso contrario, genera un botón por cada elemento en el generador
        const botonPaginado = document.createElement('a');
        botonPaginado.href = '#';
        botonPaginado.dataset.pagina = value;
        botonPaginado.textContent = value;
        botonPaginado.classList.add('siguiente', 'bg-yellow-400', 'px-4', 'py-1', 'mr-2', 'font-bold', 'mb-4', 'rounded');

        botonPaginado.onclick = () => {
            console.log(value);
            paginaActual = value;//variable global paginaActual varia
            buscarImagenes();// ir a la función pero tendrá un número de páginaACtual diferente
        }
        paginacionDiv.appendChild(botonPaginado);
    }
}

// 5. Primeros pasos para crear un paginador
// Se debe saber cuantas imagenes se tiene disponibles. Se sabe con totalHits usando la API, totalHits es dinámico
// La paginación será de acuerdo a los registros disponibles
// ¿Cuántos elementos se mostrarán? const registrosPorPagina = 40;
// ¿Cómo calcular el total de páginas? ej: 500/40=12.5 redondear hacia arriba
// parseInt(Math.ceil(500/40))=13 páginas
// Mostrar en el HTML las 13 páginas usando function *generador Iterador
// 6. Generar un Paginador con un Generador
// Viene con un iterador y nos permite identificar cuando llegamos al final. Sin el generador tendríamos que estar revisando,
//  si ya llegamos a la última página
// 7. Mostrando la cantidad de páginas del Paginador. en la function imprimirPaginador
// 8. Navegar por las páginas. cuando se da click, lea el valor y me lleve a esa página. se realiza en imprimirPaginador.
//Para navegar por las páginas se tiene que volver a consultar la API 
// para ello primero ver la documentación de la API donde dice 
// page default 1
// no todas las APIs tienen paginación, si no cuenta con ella.
// será difícil realizar la paginación
// entonces ir a la URL y colocar &page=${paginaActual}

function limpiarHTML(limpiando) {
    while (limpiando.firstChild) {//Selector global resultado 
        limpiando.removeChild(limpiando.firstChild);
    }
}
```
