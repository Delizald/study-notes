## var and let

## exports and imports (modules)

default export you choose the name:
  import person from './person.js'
  import prs from './person.js'

named export name is already defined:
  import { smth } from './something.js'
  import { smth as Smth } from './something.js'
  import { * sd bundled } from './something.js'

## classes
  a derived class must call "super" in the constructor:
  class Person extends Human {
    constructor() {
      super();
      this.name = 'David';
    }
  }
### difference between classes and prototypes
  TODO

## spread & rest operators
  spread: '...' Used to split array elements or object properties
    const newArray = {...oldArray, 1, 2,}
    const newObject = {...oldObject, newProp: 5}

  rest: Used to merge a list of function arguments into an array
    function sortArgs(...args) {
      return args.sort();
    }

## Desctructuring
  Easily extract array elements or object properties and store them in variables
  [a,b] = ['Hello', 'World'];
  console.log(a); // Hello
  
  Object destructuring:
  {name} = {name: 'David', age: '32'}
  console.log(name) // David
  console.log(age) // undefined

  ## reference types
  const person = { name: 'David'};
  const secondPerson = person // points to the same address in memory
  person.name = 'Adrian'
  console.log(secondPerson)  // 'Adrian'
  

  const person = { name: 'David'};
  const secondPerson = {...person}
  person.name = 'Adrian'
  console.log(secondPerson)  // 'David'


  ## important js array functions
    TODO
    map()
    find()
    findIndex()
    filter()
    reduce()
    concat()