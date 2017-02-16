# Transactional

```
  Transactional  <- Messenger <- Mixable
    |        |
    V        V
Record     Collection
```

Abstract base class for the Record and Collection
representing their common functionality related to 
execution of transactions, serialization, and validation on the object trees.

- as a Messenger, can send and receive events.
- can be a part of ownership tree.
- can be transactionally changed.
- notifies listeners and owner on changes.
- can be serialized to JSON.
- can be validated.

### Ownership tree

dede

## set( data, options? )

## toJSON()

## parse( data, options )

## clone( options )

## validate()

## validationError

## isValid()

## cid