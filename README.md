
- Saca en una consulta cuántos apartamentos hay en España

<pre>
<code>
use('exampleDatabases');

db.listingsAndReviews.find({
    "address.country": 'Spain'
}).count();
</code>
</pre>

- Lista los 10 primeros:
- Sólo muestra: nombre, camas, precio, government_area
- Ordenados por precio.

<pre>
<code>
use('exampleDatabases');
db.listingsAndReviews.find(
    {"address.country": 'Spain'},
    {
    name: 1, 
    beds: 1, 
    price: 1, 
    "address.government_area": 1 
    }
    )
    .sort({price: 1});
    .limit(10)
</code>
</pre>

- Queremos viajar cómodos, somos 4 personas y queremos:
- 4 camas.
- Dos cuartos de baño.

<pre>
<code>
use('exampleDatabases');
db.listingsAndReviews.find(
    {"address.country": 'Spain', beds: {$gte: 4}, bathrooms: {$gte: 2}},
    {
    name: 1, 
    beds: 1, 
    price: 1, 
    "address.government_area": 1 
    }
    )
    .sort({price: 1})
    .limit(10);
</code>
</pre>

a) Al requisito anterior, hay que añadir que nos gusta la tecnología queremos que el apartamento tenga
wifi.

b) Y bueno, un amigo se ha unido que trae un perro, así que a la query anterior tenemos que buscar que
permitan mascota Pets Allowed

<pre>
<code>
use('exampleDatabases');
db.listingsAndReviews.find(
    {amenities: {$in: ['Wifi', 'Pets allowed']}}
)
</code>
</pre>

- Estamos entre ir a Barcelona o a Portugal, los dos destinos nos valen, peeero... queremos que el precio nos salga baratito (50 $), y que tenga buen rating de reviews

<pre>
<code>
use('exampleDatabases');
db.listingsAndReviews.aggregate(
    [
        {$match: 
             {
                "$and":[
                {"$or":[
                    {"address.market": 'Barcelona'},
                    {"address.country": 'Portugal'}
                    ]   
                },
                {"price": {$lte: 50}}
                ]
             }            
        },
        {$sort: {price: 1}}     
    ]
)
</code>
</pre>

- Queremos mostrar los pisos que hay en España, y los siguiente campos:
  - Nombre.
  - De que ciudad (no queremos mostrar un objeto, solo el string con la ciudad)
  - El precio (no queremos mostrar un objeto, solo el campo de precio)

<pre>
<code>
use('exampleDatabases');
db.listingsAndReviews.find(
    {"address.country": 'Spain'},
    {
    name: 1, 
    "address.city": 1, 
    price: 1
    }
    )
    .sort({price: 1});
</code>
</pre>

- Queremos saber cuántos alojamientos hay disponibles por país.

<pre>
<code>
use('exampleDatabases');

db.listingsAndReviews.aggregate(
    [
        {$group: 
            {
                _id: "$address.country",
                count: {$sum: 1}
            }
        }
    ]
)
</code>
</pre>

- Queremos saber el precio medio de alquiler de airbnb en España.

<pre>
<code>
use('exampleDatabases');

db.listingsAndReviews.aggregate(
    [
        {$match:    
            {"address.country": 'Spain'}     
        },
        {$group: 
            {
                _id: "$address.country",
                average: {$avg: "$price"}
            }
        }
    ]
)
</code>
</pre>

- ¿Y si quisiéramos hacer como el anterior, pero sacarlo por paises?

<pre>
<code>
use('exampleDatabases');
db.listingsAndReviews.aggregate([
    {$group: 
        {
            _id: "$address.country",
            average: {$avg: "$price"}
        }
    }
])
</code>
</pre>

- Repite los mismos pasos pero agrupando también por número de habitaciones.  

<pre>
<code>
use('exampleDatabases');
db.listingsAndReviews.aggregate([
    {$group:
        {
            _id: {
                country: "$address.country",
            },
            average: {$avg: "$beds"}
        }
    }
])
</code>
</pre>

- Queremos mostrar el top 5 de apartamentos más caros en España, y sacar los siguentes campos:
  - Nombre.
  - Ciudad.
  - Amenities, pero en vez de un array, un string con todos los amenities.

<pre>
<code>
use('exampleDatabases');
db.listingsAndReviews.aggregate([
    {$match:    
        {"address.country": 'Spain'}     
    },
    {$sort: {price: -1}},
    {$limit: 5},
    {$project: 
        {
            _id: 0,
            name: 1,
            price: 1,
            city: 1,
            amenities: {
                $reduce: {
                    input: "$amenities",
                    initialValue: "",
                    in: { $concat: [ "$$value", "-", "$$this", " " ] }
                }
            }
        }
    }
])
</code>
</pre>