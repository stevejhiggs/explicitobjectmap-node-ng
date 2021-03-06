# explicit-object-mapper-ng

Map named fields from one json object to another with optional transforms. Any fields not named in the map will not be copied to the destination object. This is a hard fork from https://github.com/opentable/explicitobjectmap-node which now appears to have no maintainers.


Mappings consist of a simple javascript array containing mapping instructions:
```
[
  'simpleA', //will just copy accross the field
  'simpleB',
  'simpleC',
  'simpleD',
  {'oldname':'newname'}, //will rename the field from oldname to newname
  {	//will rename the field then run the custom tranform on the result
    srcName:'complexoldname',
    dstName:'complexnewname',
    customTransform: function (srcObj, val){
      return val.toUpperCase();
    }
  },
  {	//will rename the field then run the mapper on that value. This allows embedding mappers inside mappers
    srcName:'sourceobjectname',
    dstName:'newname',
    mapper: explicitMapper(['simpleE'])
  },
  {'deep.childA': 'baby'}, //dot notation is currently only supported when renaming fields
  function(srcObj,dstObj){
    dstObj.CustomField = 'whatever'; //post mapping function ran after all the other maps are ran
  }
]
```

## Installation
	npm install explicit-object-mapper-ng

## usage
```
const explicitObjectMapper = require('explicit-object-mapper-ng');

const mapObj =
[
  'simpleA',
  {'oldname':'myVal'},
];

const srcObj = {
  simpleA: 'alpha',
  oldname: 'changedName'
};

const mapper = explicitObjectMapper.createMapper(mapObj);
const dstObj = mapper.map(srcObj);
```

The output from the above would be:

```
{
  simpleA: 'alpha',
  changedName: 'myVal'
}
```

If an array of objects is passed in then all objects will be mapped and returned in an array.

## Speed
There is some overhead to the mapping process depending on map size and the amount of source data; this can be mitigated a little by creating the mappings ahead of time and reusing them. That said, on my machine the benchmark script, when ran on my machine runs the "basic map" benchmark in 369ms, thats 10 million maps.

## Changes

### changes in 4.0.0

* changed calling convention to make commonJs interop nicer

### changes in 3.0.0

* massive package version update
* sped up maps using dot notation by over 100%
* reduced dependancies on external packages
* now needs node >= 8
* extremely basic typescript support

### changes in 2.0.0
Now needs node > 4.2
Uses es6 features to clean up codebase

### changes in 1.0.0

#### added ability to embed mappers inside maps
We can now add mappers inside maps, for example:

```
const objectToMap = { Name: { First: 'Bob', Last: 'Smith' }};
const childMap = ['Firstname'];
const rootMap = [
    {
        srcName:'Name',
        dstName:'IncompleteName',
        mapper: childMap
    }
];
const mappedObject = rootMap.map(objectToMap); // { IncompleteName: { Firstname: 'Bob' } }
```

#### optional args to map
map can be called with an optional options variable:

	mapper.map(srcObj, {myVal: true, myOtherVal:'biscuit'});

This object is then passed into any custom mapping functions:

```
[
  'simpleA',
  {'oldname':'newname'},
  {
    srcName:'complexoldname',
    dstName:'complexnewname',
    customTransform: function (srcObj, val, options){
      return val.toUpperCase() + options.myOtherVal;
    }
  },
  function(srcObj,dstObj, options){
    dstObj.CustomField = 'whatever'; //post mapping function ran after all the other maps are ran
  }
]
```

### changes in 0.0.6

#### null values now handled correctly
Null values were still not handled properly; They will now get mapped

### changes in 0.0.5

#### falsy values now handled correctly
previously if a source value could be evaluated as false (null, 0, false) then the relevant field would not be mapped, now the field is mapped as long as the source field exists
