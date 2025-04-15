<script setup lang="ts">
import { reactive } from 'vue';
import ListItem from './ListItem.vue'
import DocumentationIcon from './icons/IconDocumentation.vue'
import IconSupport from './icons/IconSupport.vue';


// Raw data state
const rawState = reactive({
  name: 'Altura'
})

// Proxy to automatically capitalize all input
const proxyState = new Proxy(rawState, {
  get(target, prop) {
    if (prop === 'name') {
      return target[prop].toUpperCase()
    }
    return target[prop]
  }
})

function awesomeArray(arr) {
  return new Proxy(arr, {
    get(target, key, receiver) {
      if (typeof key === 'string' && key.includes(':')) {
        const [start, end] = key.split(':');
        return arr.slice(start, end)
      }
      return Reflect.get(target, key, receiver);
    },
    set(target, prop, value, receiver) {
      // TODO: Implement setter for this array
      // For example saying fruits['1:2'] = ['lemon', 'peach'] should set these values to correct positions
      return Reflect.set(target, prop, value, receiver); // This is normal behaviour
    }
  })
}

const fruits = awesomeArray(['orange', 'apple', 'banana', 'peach', 'watermelon'])

// TODO: Make it possible to set objects like this
// fruits['1:2'] = ['lemon', 'peach']
</script>

<template>
  <ListItem>
    <template #icon>
      <DocumentationIcon />
    </template>
    <template #heading>Proxy objects</template>
      A Proxy in JavaScript is an object that wraps another object (called the target) and intercepts operations performed on it, like getting or setting properties. 
      <br>
      Think of it as a middleman or a "trap layer" between your code and the actual object.
      
      Why use proxies?
      <ul>
        <li> Control access to the objects </li>
        <li> Enhance the target's behaviour </li>
      </ul>
  </ListItem>
  <ListItem>
    <template #icon>
      <DocumentationIcon />
    </template>
    <template #heading> Key concepts </template>
    <ul>
      <li> Target: The original object you're wrapping. </li>
      <li> Handler: An object with trap methods that define custom behavior for operations like get, set, delete, etc.</li>
    </ul>
  </ListItem>
  <ListItem>
    <template #icon>
      <DocumentationIcon />
    </template>
    <template #heading> Live Proxy: Input Interception </template>
    Try typing into the input below. The Proxy will intercept the input and automatically transform it to uppercase:
    <input v-model="rawState.name" class="border rounded px-2 py-1 mt-2 block" />
    <p class="mt-2">
      Proxied value: <strong>{{ proxyState.name }}</strong>
    </p>
    <p>
      It can be achieved with this simple Proxy
    </p>
    <pre>
      <code class="custom-code">
        const proxyState = new Proxy(rawState, {
          get(target, prop) {
            if (prop === 'name') {
              return target[prop].toUpperCase()
            }
            return target[prop]
          }
        })

        // To print it in uppercase
        { proxyState.name }
      </code>
    </pre>
  </ListItem>
  <ListItem>
    <template #icon>
      <DocumentationIcon />
    </template>
    <template #heading> One of the interesting use cases of proxies </template>
    <pre>
      <code class="custom-code">
        function awesomeArray(arr) {
          return new Proxy(arr, {
            get(target, key) {
              if (typeof key === 'string' && key.includes(':')) {
                const [start, end] = key.split(':');
                return arr.slice(start, end)
              }
              return target[key];
            }
          })
        }

        const fruits = awesomeArray(['orange', 'apple', 'banana', 'peach', 'watermelon'])
      </code>
      <p><code>fruits[1]</code> {{ fruits[1] }}</p>
      <p><code>fruits['0:3']</code> {{ fruits['0:3'] }}</p>
    </pre>

  </ListItem>
</template>

<style lang="css" scoped>
code{
  padding: 2px 4px;
  font-size: 90%;
  color: steelblue;
  background-color: #f8f8f2;
  border-radius: 4px;
}

i {
  color: steelblue;
}

.custom-code {
  font-family: "Fira Code", monospace;
  padding: 10px;
  border-radius: 5px;
  display: block;
  overflow-x: auto;

  color: steelblue;
  background-color: #f8f8f2;
}

button{
  border-radius: 6px;
  border: none;
  color: white;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  cursor: pointer;
  padding: 2.5px;

  background-color: steelblue;
}
</style>
