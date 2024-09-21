
# Custom Hooks en React
Los custom hooks en React te permiten reutilizar lógica de estado en diferentes componentes, lo que contribuye a que tu código sea más modular y fácil de mantener. En este tutorial aprenderás cómo crearlos, cuándo usarlos, y cómo sacarles el máximo provecho.
## 1. ¿Qué son los Custom Hooks?

Un custom hook es una función de JavaScript que sigue las reglas de los hooks de React (`useState`, `useEffect`, etc.) y encapsula lógica reutilizable. Se pueden usar para compartir lógica relacionada con el estado o los efectos sin necesidad de repetir código.

Los custom hooks siguen las mismas reglas que los hooks estándar:
- Deben comenzar con `use`.
- Deben ser llamados solo desde componentes de React o desde otros hooks.

### ¿Por qué usarlos?
Los custom hooks son útiles para encapsular y compartir lógica entre componentes. En lugar de tener código duplicado en varios componentes, puedes mover esa lógica a un custom hook y reutilizarlo fácilmente.

## 2. Creación de Custom Hooks
Ejemplo básico: Uso de `useState` en un Custom Hook

Imagina que queremos manejar un contador en varios componentes. En lugar de repetir la lógica en cada componente, podemos crear un hook personalizado:
~~~
//useCounterHook.js 
import { useState } from 'react';

function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);
  const reset = () => setCount(initialValue);

  return { count, increment, decrement, reset };
}

export default useCounter;
~~~

En este caso, useCounter encapsula la lógica del contador, y podemos reutilizarlo en cualquier componente.

## Uso del Custom Hook en Componentes
Ahora, en lugar de escribir la lógica del contador en cada componente, simplemente usamos el custom hook:
~~~
import React from 'react';
import useCounter from './useCounterHook';

function CounterComponent() {
  const { count, increment, decrement, reset } = useCounter(10);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}

export default CounterComponent;
~~~

Este enfoque permite que la lógica del contador sea reutilizable y el componente sea más limpio y fácil de leer.

## 3. Custom Hooks con useEffect
Los custom hooks también pueden manejar efectos secundarios (side effects) utilizando useEffect. Por ejemplo, un hook para manejar eventos de window podría ser útil en varios componentes.

### Ejemplo: Manejar el tamaño de la ventana
Podemos crear un custom hook que rastree el tamaño de la ventana y actualice el estado cuando el tamaño cambie:
~~~
// useWindowSizeHook.js
import { useState, useEffect } from 'react';

function useWindowSize() {
  const [windowSize, setWindowSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    const handleResize = () => {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    };

    window.addEventListener('resize', handleResize);

    // Cleanup para eliminar el evento cuando el componente se desmonta
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return windowSize;
}

export default useWindowSize;
~~~

### Uso del hook en un componente:
~~~
import React from 'react';
import useWindowSize from './useWindowSizeHook';

function WindowSizeComponent() {
  const { width, height } = useWindowSize();

  return (
    <div>
      <p>Ancho de la ventana: {width}px</p>
      <p>Alto de la ventana: {height}px</p>
    </div>
  );
}

export default WindowSizeComponent;
~~~
Este hook permite reutilizar la lógica de manejo del tamaño de la ventana en cualquier componente sin repetir código.

## 4. Custom Hooks con lógica asíncrona
Los custom hooks también pueden manejar lógica asíncrona, como la obtención de datos de una API. Aquí hay un ejemplo de cómo crear un hook personalizado para hacer solicitudes de datos:

### Ejemplo: Fetch de datos asíncrono
~~~
// useFetchHook.js
import { useState, useEffect } from 'react';

function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch(url);
        if (!response.ok) {
          throw new Error('Error en la solicitud');
        }
        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [url]);

  return { data, loading, error };
}

export default useFetch;
~~~

### Uso del hook para obtener datos:
~~~
import React from 'react';
import useFetch from './useFetchHook';

function DataFetchingComponent() {
  const { data, loading, error } = useFetch('https://jsonplaceholder.typicode.com/posts');

  if (loading) return <p>Cargando...</p>;
  if (error) return <p>Error: {error}</p>;

  return (
    <ul>
      {data.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}

export default DataFetchingComponent;
~~~
Este custom hook puede reutilizarse para cualquier solicitud de datos, simplemente cambiando la URL.

## 5. Buenas prácticas para Custom Hooks
### a) Encapsular solo la lógica necesaria
Un custom hook debe encapsular una única responsabilidad. Si empiezas a agregar demasiada lógica en un solo hook, se vuelve difícil de mantener. En lugar de esto, divide tu lógica en múltiples hooks si es necesario.
### b) Evita los efectos secundarios en hooks innecesarios
Asegúrate de que los custom hooks no provoquen efectos secundarios inesperados. Un hook debe ser predecible y fácil de usar.
### c) Reutilización y composición
Un gran beneficio de los custom hooks es la posibilidad de componerlos. Puedes usar un custom hook dentro de otro para combinar lógica compleja:
~~~
// usePostsHook.js
import useFetch from './useFetchHook';

function usePosts() {
  const { data: posts, loading, error } = useFetch('https://jsonplaceholder.typicode.com/posts');
  
  return { posts, loading, error };
}
~~~

## 6. Ejemplos avanzados
### a) Custom Hook para formularios
Uno de los casos más comunes para los custom hooks es el manejo de formularios. Aquí hay un ejemplo simple de un hook para manejar formularios:
~~~
// useFormHook.js
import { useState } from 'react';

function useForm(initialValues) {
  const [values, setValues] = useState(initialValues);

  const handleChange = (e) => {
    const { name, value } = e.target;
    setValues({
      ...values,
      [name]: value,
    });
  };

  const resetForm = () => setValues(initialValues);

  return { values, handleChange, resetForm };
}

export default useForm;
~~~

### Uso del hook en un formulario:
~~~
import React from 'react';
import useForm from './useFormHook';

function FormComponent() {
  const { values, handleChange, resetForm } = useForm({ name: '', email: '' });

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log(values);
    resetForm();
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" value={values.name} onChange={handleChange} />
      <input name="email" value={values.email} onChange={handleChange} />
      <button type="submit">Enviar</button>
    </form>
  );
}

export default FormComponent;
~~~

### b) Hook para debouncing
El "debouncing" es una técnica que limita la frecuencia de ejecución de una función. Puedes crear un hook personalizado para aplicar esta técnica:
~~~
// useDebounceHook.js
import { useState, useEffect } from 'react';

function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(handler);
  }, [value, delay]);

  return debouncedValue;
}

export default useDebounce;
~~~
Este hook es útil para evitar que se llame a una función (por ejemplo, una búsqueda) demasiado rápido.

### Uso del hook en un componente
Aquí tienes un ejemplo de cómo usar el hook `useDebounce` en un componente React. Este componente simula una búsqueda en tiempo real y utiliza el debounce para limitar las llamadas a la función de búsqueda:
~~~
import React, { useState } from 'react';
import useDebounce from './useDebounceHook'; // Asegúrate de importar el hook

function SearchComponent() {
  const [searchTerm, setSearchTerm] = useState('');
  const [results, setResults] = useState([]);
  
  // Usamos el hook useDebounce
  const debouncedSearchTerm = useDebounce(searchTerm, 300); // 300 ms de delay

  // Simulamos una búsqueda (puedes reemplazar esto con una llamada a una API)
  const performSearch = (query) => {
    if (query) {
      const simulatedResults = ['apple', 'banana', 'orange', 'grape', 'kiwi'].filter(item =>
        item.toLowerCase().includes(query.toLowerCase())
      );
      setResults(simulatedResults);
    } else {
      setResults([]);
    }
  };

  // useEffect para llamar a la búsqueda solo cuando el término de búsqueda se debouncea
  React.useEffect(() => {
    performSearch(debouncedSearchTerm);
  }, [debouncedSearchTerm]);

  return (
    <div>
      <input
        type="text"
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="Buscar..."
      />
      <ul>
        {results.map((result, index) => (
          <li key={index}>{result}</li>
        ))}
      </ul>
    </div>
  );
}

export default SearchComponent;
~~~
#### Explicación:

- Input de búsqueda: El usuario escribe en el campo de entrada, y el estado searchTerm se actualiza.
- Debounce: El hook useDebounce se utiliza para debilitar el valor de searchTerm. Solo después de que el usuario deja de escribir durante 300 ms, el valor se actualiza.
- Búsqueda: Cuando debouncedSearchTerm cambia, se ejecuta performSearch, que filtra una lista simulada de frutas en función del término de búsqueda.
- Resultados: Los resultados se muestran en una lista.

Con este enfoque, evitas hacer llamadas innecesarias a la función de búsqueda mientras el usuario escribe rápidamente.