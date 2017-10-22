# Implement.js

WIP

* Create Java-like interfaces for classes or objects
* Effortlessly and safely parse API responses
* Warnings are suppressed when process.env.NODE_ENV === 'production'

TODO

1. interface function
2. enum type
3. extend interfaces
4. unit tests
5. ensure original object reference is kept
###### What is Implement.js?
Implements is library that attempts to bring interfaces to JavaScript. Simply define an interface, and call `implements` on a class or object to ensure it implement the given interface.
```
class Hello {
    greeting = 'hello'
    wave () {}
}
const Introduction = interface({
    greeting: type('string')
    handshake: type('function')
}, { error: true })

const HelloIntroduction = implement(Introduction)(Hello) // throws an error!
```

###### Why use Implements.js?

todo

## API

### Implements
Accepts an interface and a class or object, and checks to see if the object implement the given interface
```
implement(Interface)(object|class) -> object|class
```

### Interface
Accepts an object, where all the keys are Type objects, and returns an Interface. The Interface is to be used by Implements.
```
interface(object[, options]) -> Interface
```
Options
```
{
    // when true, errors and warnings are triggered when properties other than those on the interface are found, is suppressed if trim is set to true - default: false
    strict: true,

    // remove methods that don’t match the interface - default: false
    trim: true,

    // throws an error when interface isn’t implemented - default: false
    error: true,

    // warns when interface isn’t implemented, provides stack trace - default: true
    warn: false,

    // accepts an interface to extend, the new interface must also implement the extended interface
    extend: Interface
}
```

### Type
Accepts a string matching any [JavaScript types](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof#Description), plus ‘array’.

If the string ‘array’ is passed, a second argument can be passed denoting the type of the elements of the array, if none is passed then the types of the elements will not be checked. The second argument should be an array containing Type or Interface. Typed arrays can be empty.

If the string ‘object’ is passed, a second argument can be passed denoting an interface for the object, if none is passed then the properties of the object will not be checked. The second argument should be an Interface. ‘object’ types match the options of the given interface.
```
type(string[, Array<type|interface>|interface]) -> Type
```

## Examples

##### Class Example:
```
import implement, { interface, type } from ‘implement’

const Passenger = interface({
    name: type(‘string’),
    height: type(‘number’)
})

const ChildPassenger = interface({
    hasBabySeat: type(‘boolean’)
}, {
    extend: Passenger
})

const Car = interface({
    speed: type(’number’),
    passengers: type(‘array’, [type('object', Passenger), type('object', ChildPassenger)]),
    beep: type(‘function’)
}, {
    error: true
})

const MyCar = implement(Car)(class {
    speed = 0
    passengers = []

    beep () {}
})

// throws error
const OtherCar = implement(Car)(class {
    speed = 0
})
```

##### Refactoring API response:
```
import { store } from ‘../store’
import { fetchUsers } from ‘../services/userService’
import implement, { interface, type } from ‘implement’

const User = interface({
    name: type(‘string’),
    id: type(‘number’)
}, {
    trim: true
})

const Users = interface({
    users: type(‘array’, [type('object', User)])
}, {
    trim: true
})

const ErrorRes = interface({
    message: type(‘string’),
    code: type(‘number’)
})


const updateUsers = () => dispatch => {
    dispatch(fetchUsers())

    fetchUsers().then(res => {
        const MyUsers = implement(Users)(res)
        store.dispatch(updateUsersSuccess(MyUsers))
    })
    .catch(err => {
        const MyErrorRes = implement(ErrorRes)(err)
        store.dispatch(updateUsersError(MyErrorRes))
    })
}
```

##### With Redux reducers:
```
import { fetchUsers } from ‘../services/userService’
import implement, { interface, type } from ‘implement’

const SomeAction = interface({
    some: type(‘string’),
    other: type(‘number’)
})

const initialState = {
    some: ‘thing’,
    other: 0
}

export default (state = initialState, action) => {
    switch(action.type) {
        case SOME_ACTION:
            return implement(SomeAction)({
                …state,
                …action.payload
            })

        default:
            return state
    }
}
```