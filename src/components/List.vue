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

/**
 * ARRAY TOPIC
 */

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


/**
 * String builder topic
 */

 function commandBuilder(path = []) {
  return new Proxy(() => {}, {
    get(_, prop) {
      if (prop === 'run') {
        return () => {
          return path.map(part => {
            if (typeof part === 'function') {
              return part(); // resolve dynamic parts
            }
            return part;
          }).join(' ');
        };
      }

      return commandBuilder([...path, prop]);
    },
    apply(_, __, args) {
      const last = path.at(-1);
      // Convert .message("Hello") to: message -m "Hello"
      if (typeof last === 'string') {
        const flag = `-${last[0]}`; // simplistic: 'message' → '-m'
        const quoted = args.map(arg => `"${arg}"`).join(' ');
        const formatter = () => `${flag} ${quoted}`;
        return commandBuilder([...path.slice(0, -1), formatter]);
      }

      // fallback (shouldn't usually happen)
      return commandBuilder(path);
    }
  });
}

const cli = commandBuilder();


// TODO: Build custom string builder using proxies
// function classNameBuilder(parts = []) {
//
// }

// const $c = classNameBuilder();


/**
 * 🤡 Example
 */

function secretFunction(x) {
  return `You passed ${x}`;
}

let callCount = 0;

const trollProxy = new Proxy(secretFunction, {
  apply(target, thisArg, args) {
    callCount++;
    console.log(`\nCalling function ${callCount} time`);
    if (callCount % 3 === 0) {
      console.log('The function is tired. It returns nonsense.');
      return '🤪 No idea what you asked!\n';
    }
    return target.apply(thisArg, args);
  },

  get(target, prop) {
    if (prop === 'selfDestruct') {
      console.log('\nGoodbye cruel world!');
      delete trollProxy.apply;
      return '💥 Boom. Gone.\n';
    }
    return Reflect.get(target, prop);
  },

  has(target, prop) {
    const result = Math.random() < 0.5;
    console.log(`\nDoes "${prop}" exist? ${result}`);
    return result;
  },

  deleteProperty(target, prop) {
    const blocked = Math.random() < 0.3;
    if (blocked) {
      console.log(`\nAccess denied: can't delete "${prop}"`);
      return true;
    }
    console.log(`\nDeleted "${prop}"\n`);
    return Reflect.deleteProperty(target, prop);
  }
});

console.log(trollProxy('Hello'));
console.log(trollProxy('Salut'));
console.log(trollProxy('Privet'));

console.log('apply' in trollProxy);

console.log(trollProxy.selfDestruct);

delete trollProxy.name;


// TODO: Implement proxy that will have secured fields that you cannot delete from the object
// Also forbid to see salary if role is not accountant
console.log('\n--------------------\n')
const securedFields = ['name', 'email'];

const sampleObject = {
  id: 123,
  name: 'John Doe',
  email: 'johndoe@altura.io',
  salary: 100000,
  age: 32,
}

// proxy should look like
const userProxy = new Proxy(sampleObject, {
  // Prevent deletion of secured fields
  deleteProperty(target, prop) {
    return Reflect.deleteProperty(target, prop);
  },
  // Protect salary unless role is accountant
  get(target, prop, receiver) {
    return Reflect.get(target, prop, receiver);
  }
})

// userProxy.all('accountant') // should return whole object if role is accountant or everything except salary if any other role
// delete userProxy.name // should fail 
// delete UserProxy.age // should not fail 
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
  <ListItem>
    <template #icon>
      <DocumentationIcon />
    </template>
    <template #heading> And it's not limited to that! </template>
    Below you can see how using 'apply' method in proxies we can build strings while not using hardcoded strings.
    <pre>
      <code class="custom-code">
        function commandBuilder(path = []) {
          return new Proxy(() => {}, {
            get(_, prop) {
              if (prop === 'run') {
                return () => {
                  return path.map(part => {
                    if (typeof part === 'function') {
                      return part(); // resolve dynamic parts
                    }
                    return part;
                  }).join(' ');
                };
              }

              return commandBuilder([...path, prop]);
            },
            apply(_, __, args) {
              const last = path.at(-1);
              // Convert .message("Hello") to: message -m "Hello"
              if (typeof last === 'string') {
                const flag = `-${last[0]}`; // simplistic: 'message' → '-m'
                const quoted = args.map(arg => `"${arg}"`).join(' ');
                const formatter = () => `${flag} ${quoted}`;
                return commandBuilder([...path.slice(0, -1), formatter]);
              }

              // fallback (shouldn't usually happen)
              return commandBuilder(path);
            }
          });
        }

        const cli = commandBuilder();
      </code>
    </pre>
    <code>cli.git.commit.message("Initial commit").run()</code> {{ cli.git.commit.message("Initial commit").run() }}
    <br>
    Now, can you make something simple?<br>
    Can you write a Proxy that will build a string in the same way, but just simply add '-' between every word?
    <code>$c.cool.class.name()</code> should resolve to <code>cool-class-name</code> for example
  </ListItem>
  <ListItem>
    <template #icon>
      <DocumentationIcon />
    </template>
    <template #heading> Let's talk about magic now </template>
    <p>Have you ever experienced any weird behaviour while working with JavaScript? If so, then maybe it was not magic, it could've been a troll proxy! <i>(just joking)</i></p>
    <pre>
      <code class="custom-code">
        function secretFunction(x) {
          return `You passed ${x}`;
        }

        let callCount = 0;

        const trollProxy = new Proxy(secretFunction, {
          apply(target, thisArg, args) {
            callCount++;
            console.log(`Calling function ${callCount} time`);
            if (callCount % 3 === 0) {
              console.log('The function is tired. It returns nonsense.');
              return '🤪 No idea what you asked!';
            }
            return target.apply(thisArg, args);
          },

          get(target, prop) {
            if (prop === 'selfDestruct') {
              console.log('Goodbye cruel world!');
              delete trollProxy.apply;
              return () => '💥 Boom. Gone.';
            }
            return Reflect.get(target, prop);
          },

          has(target, prop) {
            const result = Math.random() < 0.5;
            console.log(`Does "${prop}" exist? ${result}`);
            return result;
          },

          deleteProperty(target, prop) {
            const blocked = Math.random() < 0.3;
            if (blocked) {
              console.log(`Access denied: can't delete "${prop}"`);
              return false;
            }
            console.log(`Deleted "${prop}"`);
            return Reflect.deleteProperty(target, prop);
          }
        });


        console.log(trollProxy('Hello'));
        console.log(trollProxy('Salut'));
        console.log(trollProxy('Privet'));

        console.log('apply' in trollProxy);

        console.log(trollProxy.selfDestruct);

        delete trollProxy.name;
      </code>
    </pre>
    <p>
      Now please implement a proxy where user is already defined and you need to overwrite validations for get and delete on this user props
    </p>
    <pre>
      <code class="custom-code">
        const securedFields = ['name', 'email'];
        const sampleObject = {
          id: 123,
          name: 'John Doe',
          email: 'johndoe@altura.io',
          salary: 100000,
          age: 32,
        }

        // proxy should look like
        const userProxy = new Proxy(sampleObject, {
          // Prevent deletion of secured fields
          deleteProperty(target, prop) {
            return Reflect.deleteProperty(target, prop);
          },
          // Protect salary unless role is accountant
          get(target, prop, receiver) {
            return Reflect.get(target, prop, receiver);
          }
        })

        // userProxy.all('accountant') // should return whole object if role is accountant or everything except salary if any other role
        // delete userProxy.name // should fail 
        // delete UserProxy.age // should not fail 
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
