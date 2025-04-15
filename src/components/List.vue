<script setup lang="ts">
import ListItem from './ListItem.vue'
import DocumentationIcon from './icons/IconDocumentation.vue'
import IconSupport from './icons/IconSupport.vue';

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
    <pre>
        <code class="custom-code">
          const target = {
            message: "Hello world"
          };

          const handler = {
            get(obj, prop) {
              if (prop === "message") {
                return obj[prop] + " 👋";
              }
              return obj[prop];
            }
          };

          const proxy = new Proxy(target, handler);

          console.log(proxy.message); // Hello world 👋
        </code>
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
