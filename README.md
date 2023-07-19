# USING MONGODB IN GO

## Introduction to MongoDB

MongoDB is an open-source, cross-platform, distributed document database. MongoDB is developed by MongoDB Inc. and categorized as a NoSQL database.

---

#### GET STARTED

- [connect mongo to go](#insertOne)

#### INSERT DOCUMENTS

- [InsertOne](#insertOne)
- [InsertMany](#insertMany)

#### SELECT DOCUMENTS

- [FindOne](#findOne)
- [Find](#find)

#### COMPARESION QUERY OPERATORS

- [$eq: Equal To Operator](#formatting_output)
- [$lt: Less Than Operator](#declaring_functions)
- [$lte: Less Than or Equal To Operator](#functions_and_variable_scope)
- [$gt: Greater Than Operator](#error_values)
- [$gte: Greater Than or Equal To Operator](#copies_of_the_arguments)
- [$ne: Not Equal To Operator](#pointers)
- [$in: In Operator](#pointer_types)
- [$nin: Not In Operator](#using_pointers_with_functions)

#### LOGICAL QUERY OPERATORS

- [$and: Logical AND Opeartor](#formatting_output)
- [$or: Logical OR Operator](#declaring_functions)
- [$not: Logical NOT Operator](#functions_and_variable_scope)
- [$nor: Logical NOR Operator](#error_values)

#### ELEMENT QUERY OPERATORS

- [$exists](#formatting_output)
- [$type](#declaring_functions)

#### ARRAY QUERY OPERATORS

- [$size](#formatting_output)
- [$elemMatch](#functions_and_variable_scope)

#### SORTING & LIMITING

- [sort(): Sorting documents](#formatting_output)
- [limit(): Limiting documents](#declaring_functions)

#### UPDATEING DOCUMENTS

- [updateOne: Update one Document](#formatting_output)
- [updateMany: Update Multiple Documents](#declaring_functions)
- [$inc: Increase / Decrease Field Value](#functions_and_variable_scope)
- [$min: Update Field Value](#error_values)
- [$max: Update Field Value](#copies_of_the_arguments)
- [$mul: Mutiply Field By a Number](#pointers)
- [$unset: Remove Fields](#pointer_types)
- [$rename: Rename Fields](#using_pointers_with_functions)

#### DELETING DOCUMENTS

- [deleteOne](#formatting_output)
- [deleteMany](#declaring_functions)

#### AGGREGATION

- [Aggregation Pipeline](#formatting_output)
- [$avg](#declaring_functions)
- [$count](#functions_and_variable_scope)
- [$sum](#error_values)
- [$max](#copies_of_the_arguments)
- [$min](#pointers)

#### INDECXES

- [MongoDB Indexes](#formatting_output)
- [Create Index](#declaring_functions)
- [Unique Index](#functions_and_variable_scope)
- [Compound index](#error_values)
- [Drop Index](#copies_of_the_arguments)

---

<div id="connect_mongo_to_go" />

## Connect mongo to go

```go
import (
	"context"
	"fmt"
	"log"
	"time"

	"go.mongodb.org/mongo-driver/bson"
	"go.mongodb.org/mongo-driver/bson/primitive"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
)

const (
	mongoURI           = "mongodb://localhost:27017"
	dbName             = "bookDB"
	collectionName     = "books"
	idFieldName        = "_id"
	prodDB             = "prodDB"
	prodCollectionName = "products"
)

var (
	ctx            = context.Background()
	collection     *mongo.Collection
	prodCollection *mongo.Collection
)

type User struct {
	Name      string    `bson:"name,omitempty"`
	Email     string    `bson:"email,omitempty"`
	CreatedAt time.Time `bson:"created_at,omitempty"`
	UpdatedAt time.Time `bson:"updated_at,omitempty"`
}

type Product struct {
	Name  string
	Price int
}

func init() {
	client, err := mongo.NewClient(options.Client().ApplyURI(mongoURI))
	if err != nil {
		log.Fatal(err)
	}

	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer cancel()

	err = client.Connect(ctx)
	if err != nil {
		log.Fatal(err)
	}

	collection = client.Database(dbName).Collection(collectionName)
	prodCollection = client.Database(prodDB).Collection(prodCollectionName)
}
```

---

<div id="insertOne" />

## InsertOne

The insertOne() method allows you to insert a single document into a collection.

The insertOne() method returns a document that contains the following fields:

- acknowledged is a boolean value. It is set to true if the insert executed with write concern or false if the write concern was disabled.

- insertedId stores the value of \_id field of the inserted document.

Note that if the collection does not exist, the insertOne() method will also create the collection and insert the document into it.

If you not provided a \_id mongoDB automatically added the \_id field and assigned it a unique ObjectId value.

```go
func main() {
      bookOne := Book{Title: "BookOne", Content: "lorem ipsum dolor sit amet, consectetur adipis"}
      createdDoc, err := CreateBook(&bookOne)
      if err != nil {
            log.Fatal(err)
      }
      // resturn a insertedID
      fmt.Println(createdDoc.InsertedID)
      // result is : ObjectID("64591e8ee3dfdefd673f52fb")
}

func CreateBook(book *Book) (*mongo.InsertOneResult, error) {
	book.CreatedAt = time.Now()
	book.UpdatedAt = time.Now()

	reuslt, err := collection.InsertOne(ctx, book)
	if err != nil {
		return nil, fmt.Errorf("failed to insert user: %w", err)
	}

	return reuslt, nil
}
```

---

<div id="insertMany" />

## InsertMany

The insertMany() allows you to insert multiple documents into a collection. Here is the syntax of the insertMany() method:

The first argument is an array of documents that you want to insert into the collection.

Second you can provide options for example you can provide the ordered which is a boolean value that determines whether MongoDB should perform an ordered or unordered insert.

In this example we use insertMany() to create multiple document at same time also provide option order

```go
func main() {
    bookOne := Book{Title: "BookOne", Content: "lorem ipsum dolor sit amet, consectetur adipis 1"}
	bookTwo := Book{Title: "BookOne", Content: "lorem ipsum dolor sit amet, consectetur adipis 2"}
	bookThree := Book{Title: "BookOne", Content: "lorem ipsum dolor sit amet, consectetur adipis 3"}
	books := []interface{}{bookOne, bookTwo, bookThree}
	createdDocs, err := CreateBooks(&books)
	if err != nil {
		log.Fatal(err)
	}
	// resturn a insertedIDs
	fmt.Println(createdDocs.InsertedIDs...)
    // result is : ObjectID("6459277549999f530f26a75b") ObjectID("6459277549999f530f26a75c") ...
}

func CreateBooks(books *[]interface{}) (*mongo.InsertManyResult, error) {
	options := options.InsertMany().SetOrdered(true)
	result, err := collection.InsertMany(ctx, *books, options)
	if err != nil {
		return nil, err
	}
	return result, nil
}

```

---

<div id="findOne" />

## FindOne

The findOne() returns a single document from a collection that satisfies the specified condition.

The findOne() accepts two optional arguments: query and projection.

- The query is a document that specifies the selection criteria.

- The projection is a document that specifies the fields in the matching document that you want to return.

If you omit the query, the findOne() returns the first document in the collection according to the natural order which is the order of documents on the disk.

If you don’t pass the projection argument, then findOne() will include all fields in the matching documents.

To specify whether a field should be included in the returned document, you use the following format:

If the value is true or 1, MongoDB will include the field in the returned document. In case the value is false or 0, MongoDB won’t include it.

By default, MongoDB always includes the \_id field in the returned documents. To suppress it, you need to explicitly specify \_id: 0 in the projection argument.

```go
func main() {
    // Id book one in db : 645927692c02dcc2b96ba1e6
    foundBook, err := findBook("645927692c02dcc2b96ba1e6")
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Read book with ID %v: %+v\n", "645927692c02dcc2b96ba1e6", *foundBook)
      // result is : {
      // Title:BookOne
      // Content:lorem ipsum dolor sit amet, consectetur adipis 1
      // CreatedAt:0001-01-01 00:00:00 +0000 UTC
      // UpdatedAt:0001-01-01 00:00:00 +0000 UTC}

      // Id not included because we set projection(_id:0)
}

func findBook(id string) (*Book, error) {
	var book Book

	objectID, err := primitive.ObjectIDFromHex(id)
	if err != nil {
		return nil, fmt.Errorf("failed to read book: %w", err)
	}

	query := bson.M{"_id": objectID}
	projection := bson.M{"_id": 0}

	opts := options.FindOne().SetProjection(projection)

	err = collection.FindOne(ctx, query, opts).Decode(&book)
	if err != nil {
		if err == mongo.ErrNoDocuments {
			return nil, fmt.Errorf("book not found")
		}
		return nil, fmt.Errorf("failed to read book: %w", err)
	}

	return &book, nil
}

```

---

<div id="find" />

## Find

The find() method finds the documents that satisfy a specified condition and returns a cursor to the matching documents.

The following shows the syntax of the find() method:

Similar to the findOne() method, the find() method accepts two optional arguments.

- query

- projection

```go
func main() {
    foundBooks, err := findBooks()
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("books: ", *foundBooks)
      // result is : [{
      // Title:BookOne
      // Content:lorem ipsum dolor sit amet, consectetur adipis 1
      // CreatedAt:0001-01-01 00:00:00 +0000 UTC
      // UpdatedAt:0001-01-01 00:00:00 +0000 UTC} ... ]

      // Id not included because we set projection(_id:0)
}

func findBooks() (*[]Book, error) {
	var books []Book

	query := bson.M{}
	projection := bson.M{"_id": 0}

	opts := options.Find().SetProjection(projection)

	cur, err := collection.Find(ctx, query, opts)
	if err != nil {
		if err == mongo.ErrNoDocuments {
			return nil, fmt.Errorf("book not found")
		}
		return nil, fmt.Errorf("failed to read book: %w", err)
	}

	for cur.Next(ctx) {
		var book Book
		err := cur.Decode(&book)
		if err != nil {
			return nil, fmt.Errorf("failed to read book: %w", err)
		}
		books = append(books, book)
	}

	return &books, nil
}

```

---

## $eq: Equal To Operator

The $eq operator is a comparison query operator that allows you to match documents where the value of a field equals a specified value.

In this example we check all the books wihch is titles equal to BookOne and as we know already Find() send back a cursor and for get all data we should loop through the books

First inject some data to product data base

```go
func main() {
	prodOne := Product{Name: "xPhone", Price: 799}
	prodTwo := Product{Name: "xTablet", Price: 899}
	prodThree := Product{Name: "SmartTablet", Price: 899}
	prodFour := Product{Name: "SmartPad", Price: 699}
	prodFive := Product{Name: "SmartPhone", Price: 599}

	products := []interface{}{prodOne, prodTwo, prodThree, prodFour, prodFive}

	prodCollection.InsertMany(ctx, products)
}
```

<br>

Now let find products with price equal to 899

```go
func main() {

	result, err := equalTo()

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*result)
	// result is : all products which is prices equal to 899
	// [{xTablet 899} {SmartTablet 899}]

}

func equalTo() (*[]Product, error) {
	var products []Product

	filter := bson.M{"price": bson.M{"$eq": 899}}
	cur, err := prodCollection.Find(ctx, filter)
	if err != nil {
		return nil, err
	}

	for cur.Next(ctx) {
		var product Product
		if err := cur.Decode(&product); err != nil {
			return nil, err
		}
		products = append(products, product)
	}

	if err := cur.Err(); err != nil {
		return nil, err
	}

	return &products, nil
}

```

---

<div id="less_than_operator" />

## $lt: Less Than Operator

The $lt operator is a comparison query operator that allows you to select the documents where the value of a field is less than a specified value.

lets find products where price is less than 799

```go
func main() {

	result, err := lessThan()

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*result)
	// result is : all products where price is less than 799
	// [{SmartPad 699} {SmartPhone 599}]

}

func lessThan() (*[]Product, error) {
	var products []Product

	filter := bson.M{"price": bson.M{"$lt": 799}}
	cur, err := prodCollection.Find(ctx, filter)
	if err != nil {
		return nil, err
	}

	for cur.Next(ctx) {
		var product Product
		if err := cur.Decode(&product); err != nil {
			return nil, err
		}
		products = append(products, product)
	}

	if err := cur.Err(); err != nil {
		return nil, err
	}

	return &products, nil
}
```

---

## $lte: Less Than or Equal To Operator

The $lte is a comparison query operator that allows you to select documents where the value of a field is less than or equal to ( <= ) a specified value.

In this example we find the products where prices is less than or equal to 699

```go
func main() {

	result, err := lessEqual()

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*result)
	// result is : all products where price is less or equal to 699
	// [{SmartPad 699} {SmartPhone 599}]

}

func lessEqual() (*[]Product, error) {
	var products []Product

	filter := bson.M{"price": bson.M{"$lte": 699}}
	cur, err := prodCollection.Find(ctx, filter)
	if err != nil {
		return nil, err
	}

	for cur.Next(ctx) {
		var product Product
		if err := cur.Decode(&product); err != nil {
			return nil, err
		}
		products = append(products, product)
	}

	if err := cur.Err(); err != nil {
		return nil, err
	}

	return &products, nil
}
```

---

## $gt: Greater Than Operator

The $gt operator is a comparison query operator that allows you to select documents where the value of a field is greater than (>) a specified value.

```go
func main() {

	result, err := greaterThan()

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*result)
	// result is : all products where price is greater than 699
	// [{xPhone 799} {xTablet 899} {SmartTablet 899}]

}

func greaterThan() (*[]Product, error) {
	var products []Product

	filter := bson.M{"price": bson.M{"$gt": 699}}
	cur, err := prodCollection.Find(ctx, filter)
	if err != nil {
		return nil, err
	}

	for cur.Next(ctx) {
		var product Product
		if err := cur.Decode(&product); err != nil {
			return nil, err
		}
		products = append(products, product)
	}

	if err := cur.Err(); err != nil {
		return nil, err
	}

	return &products, nil
}
```

---

## $gte: Greater Than or Equal To Operator

The $gte is a comparison query operator that allows you to select documents where a value of a field is greater than or equal to ( i.e. >=) a specified value.

In this example we find the products where price is less or euqal to 699

```go
func main() {

	result, err := greaterEqual()

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*result)
	// result is : all products where price is greater or equal to 699
	// [{xPhone 799} {xTablet 899} {SmartTablet 899} {SmartPad 699}]

}

func greaterEqual() (*[]Product, error) {
	var products []Product

	filter := bson.M{"price": bson.M{"$gte": 699}}
	cur, err := prodCollection.Find(ctx, filter)
	if err != nil {
		return nil, err
	}

	for cur.Next(ctx) {
		var product Product
		if err := cur.Decode(&product); err != nil {
			return nil, err
		}
		products = append(products, product)
	}

	if err := cur.Err(); err != nil {
		return nil, err
	}

	return &products, nil
}
```

---

## $ne: Not Equal To Operator

The $ne is a comparison query operator that allows you to select documents where the value of a filed is not equal to a specified value. It also includes documents that don’t contain the field.

```go
func main() {

	result, err := notEqual()

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*result)
	// result is : all products where price is not equal to 699
	// [{xPhone 799} {xTablet 899} {SmartTablet 899} {SmartPhone 599}]

}

func notEqual() (*[]Product, error) {
	var products []Product

	filter := bson.M{"price": bson.M{"$ne": 699}}
	cur, err := prodCollection.Find(ctx, filter)
	if err != nil {
		return nil, err
	}

	for cur.Next(ctx) {
		var product Product
		if err := cur.Decode(&product); err != nil {
			return nil, err
		}
		products = append(products, product)
	}

	if err := cur.Err(); err != nil {
		return nil, err
	}

	return &products, nil
}
```

---

## $in: In Operator

The $in is a comparison query operator that allows you to select documents where the value of a field is equal to any value in an array.

```go
unc main() {

	result, err := hasTheseValues()

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*result)
	// result is : all products where price is not equal to 699
	// [{xTablet 899} {SmartTablet 899} {SmartPad 699}]

}

func hasTheseValues() (*[]Product, error) {
	var products []Product

	filter := bson.M{"price": bson.M{"$in": []int{699, 899}}}
	cur, err := prodCollection.Find(ctx, filter)
	if err != nil {
		return nil, err
	}

	for cur.Next(ctx) {
		var product Product
		if err := cur.Decode(&product); err != nil {
			return nil, err
		}
		products = append(products, product)
	}

	if err := cur.Err(); err != nil {
		return nil, err
	}

	return &products, nil
}
```

---

## $nin: Not In Operator

The $nin is a query comparison operator that allows you to find documents where:

- the value of the field is not equal to any value in an array
- the field does not exist.

```go
func main() {

	result, err := hasNotTheseValues()

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*result)
	// result is : all products where price is not equal to 699
	// [{xPhone 799} {SmartPhone 599}]

}

func hasNotTheseValues() (*[]Product, error) {
	var products []Product

	filter := bson.M{"price": bson.M{"$nin": []int{699, 899}}}
	cur, err := prodCollection.Find(ctx, filter)
	if err != nil {
		return nil, err
	}

	for cur.Next(ctx) {
		var product Product
		if err := cur.Decode(&product); err != nil {
			return nil, err
		}
		products = append(products, product)
	}

	if err := cur.Err(); err != nil {
		return nil, err
	}

	return &products, nil
}
```

---

## $and: Logical AND Opeartor

The $and is a logical query operator that allows you to carry a logical AND operation on an array of one or more expressions.

The $and operator returns true if all expressions evaluate to true.

The $and operator stops evaluating the remaining expressions as soon as it finds an expression that evaluates to false. This feature is called **short-circuit evaluation**.

```go
func main() {

	result, err := logicalAnd()

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*result)
	// result is : all products where price is 599 and name is SmartPhone
	// [{SmartPhone 599}]

}

func logicalAnd() (*[]Product, error) {
	var products []Product

	filter := bson.M{"$and": []bson.M{
		{"price": 599},
		{"name": "SmartPhone"},
	}}
	cur, err := prodCollection.Find(ctx, filter)
	if err != nil {
		return nil, err
	}

	for cur.Next(ctx) {
		var product Product
		if err := cur.Decode(&product); err != nil {
			return nil, err
		}
		products = append(products, product)
	}

	if err := cur.Err(); err != nil {
		return nil, err
	}

	return &products, nil
}
```

---

## $or: Logical OR Operator

The $or is a logical query operator that carries a logical OR operation on an array of one or more expressions and selects the documents that satisfy at least one expression.

```go
func main() {

	result, err := logicalOr()

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*result)
	// result is : all products where price is 699 or name is SmartPhone
	// [{SmartPad 699} {SmartPhone 599}]

}

func logicalOr() (*[]Product, error) {
	var products []Product

	filter := bson.M{"$or": []bson.M{
		{"price": 699},
		{"name": "SmartPhone"},
	}}
	cur, err := prodCollection.Find(ctx, filter)
	if err != nil {
		return nil, err
	}

	for cur.Next(ctx) {
		var product Product
		if err := cur.Decode(&product); err != nil {
			return nil, err
		}
		products = append(products, product)
	}

	if err := cur.Err(); err != nil {
		return nil, err
	}

	return &products, nil
}
```

---

## $not: Logical NOT Operator

The $not operator is a logical query operator that performs a logical NOT operation on a specified **expression** and selects documents that do not match the **expression**. This includes the documents that do not contain the field.

```go
func main() {

	result, err := logicalNot()

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*result)
	// result is : all products where price is 699 or name is SmartPhone
	// [{SmartPhone 599}]

}

func logicalNot() (*[]Product, error) {
	var products []Product

	filter := bson.M{"price": bson.M{"$not": bson.M{"$gte": 699}}}

	cur, err := prodCollection.Find(ctx, filter)
	if err != nil {
		return nil, err
	}

	for cur.Next(ctx) {
		var product Product
		if err := cur.Decode(&product); err != nil {
			return nil, err
		}
		products = append(products, product)
	}

	if err := cur.Err(); err != nil {
		return nil, err
	}

	return &products, nil
}
```

---

## $nor: Logical NOR Operator

The $nor is a logical query operator that allows you to perform a logical NOR operation on a list of one or more query expressions and selects documents that fail all the query expressions.

```go
func main() {

	result, err := logicalNotOR()

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*result)
	// result is : all products where price is 699 or name is SmartPhone
	// [{xPhone 799} {xTablet 899} {SmartTablet 899}]

}

func logicalNotOR() (*[]Product, error) {
	var products []Product

	filter := bson.M{
		"$nor": []bson.M{
			{"price": bson.M{"$lte": 699}},
			{"name": "SmartPhone"},
		},
	}

	cur, err := prodCollection.Find(ctx, filter)
	if err != nil {
		return nil, err
	}

	for cur.Next(ctx) {
		var product Product
		if err := cur.Decode(&product); err != nil {
			return nil, err
		}
		products = append(products, product)
	}

	if err := cur.Err(); err != nil {
		return nil, err
	}

	return &products, nil
}
```

---

## $exists

The `$exists` is an element query operator
When the boolean_value is true, the $exists operator matches the documents that contain the field with any value including null.

$exists operator matches the documents that have the price field including the non-null and null values.

```go
func main() {

	result, err := exists()

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*result)
	// result is : all products where price is not null
	//  [{xPhone 799} {xTablet 899} {SmartTablet 899} {SmartPad 699} {SmartPhone 599}]
	// In this example contain all products becuase all prices have value

}

func exists() (*[]Product, error) {
	var products []Product

	filter := bson.M{"price": bson.M{"$exists": true}}

	cur, err := prodCollection.Find(ctx, filter)
	if err != nil {
		return nil, err
	}

	for cur.Next(ctx) {
		var product Product
		if err := cur.Decode(&product); err != nil {
			return nil, err
		}
		products = append(products, product)
	}

	if err := cur.Err(); err != nil {
		return nil, err
	}

	return &products, nil
}

```

---

## $size

The $size is an array query operator that allows you to select documents that have an array containing a specified number of elements.

```go
func main() {

	result, err := size()

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*result)
	// result is : all products where price contain 2 lements
	// which is in this example is null array

}

func size() (*[]Product, error) {
	var products []Product

	filter := bson.M{"price": bson.M{"$size": 1}}

	cur, err := prodCollection.Find(ctx, filter)
	if err != nil {
		return nil, err
	}

	for cur.Next(ctx) {
		var product Product
		if err := cur.Decode(&product); err != nil {
			return nil, err
		}
		products = append(products, product)
	}

	if err := cur.Err(); err != nil {
		return nil, err
	}

	return &products, nil
}
```

---

## $elemMatch

The $elemMatch is an array query operator that matches documents that contain an array field and the array field has at least one element that satisfies all the specified queries.

```go
func main() {

	result, err := elemMatch()

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*result)
	// result is : check array price and return all products where at least
	// one of the price above the 200
	// which is in this example is null array

}

func elemMatch() (*[]Product, error) {
	var products []Product

	filter := bson.M{"price": bson.M{"$elemMatch": bson.M{"$gt": 200}}}

	cur, err := prodCollection.Find(ctx, filter)
	if err != nil {
		return nil, err
	}

	for cur.Next(ctx) {
		var product Product
		if err := cur.Decode(&product); err != nil {
			return nil, err
		}
		products = append(products, product)
	}

	if err := cur.Err(); err != nil {
		return nil, err
	}

	return &products, nil
}
```

---

## sort(): Sorting documents

The sort() method allows you to sort the matching documents by one or more fields (field1, field2, …) in ascending or descending order.

The order takes two values: 1 and -1. If you specify { field: 1 }, the sort() will sort the matching documents by the field in ascending order:

```go
func main() {

	result, err := sort()

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*result)
	// result is : get all products where price greater than 600 and show just name and price
	// then sort with price accending
	// one of the price above the 200
	// which is in this example is null array

}

func sort() (*[]Product, error) {
	var products []Product

	filter := bson.M{"price": bson.M{"$gt": 200}}
	projection := bson.M{"_id": 0, "name": 1, "price": 1}

	opts := options.Find().SetProjection(projection).SetSort(bson.M{"price": 1})

	cur, err := prodCollection.Find(ctx, filter, opts)
	if err != nil {
		return nil, err
	}

	for cur.Next(ctx) {
		var product Product
		if err := cur.Decode(&product); err != nil {
			return nil, err
		}
		products = append(products, product)
	}

	if err := cur.Err(); err != nil {
		return nil, err
	}

	return &products, nil
}
```

---

## limit(): Limiting documents

The find() method may return a lot of documents to the application. Typically, the application may not need that many documents.

To limit the number of returned documents, you use the limit() method:

The skip(offset) requires the MongoDB server to scan from the beginning of the result set before starting to return the documents. When the (offset) increases, the skip() will become slower.

```go
func main() {

	result, err := sort()

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*result)
	// result is : get all products where price greater than 600 and show just name and price
	// then sort with price accending
	// one of the price above the 200
	// which is in this example is null array

}

func sort() (*[]Product, error) {
	var products []Product

	filter := bson.M{"price": bson.M{"$gt": 200}}
	projection := bson.M{"_id": 0, "name": 1, "price": 1}

	opts := options.Find().SetProjection(projection).SetSort(bson.M{"price": 1}).SetLimit(50).SetSkip(0)

	cur, err := prodCollection.Find(ctx, filter, opts)
	if err != nil {
		return nil, err
	}

	for cur.Next(ctx) {
		var product Product
		if err := cur.Decode(&product); err != nil {
			return nil, err
		}
		products = append(products, product)
	}

	if err := cur.Err(); err != nil {
		return nil, err
	}

	return &products, nil
}
```

---

## findOneAndUpdate: Update one Document

The findOneAndUpdate() method allows you to update a single document that satisfies a condition.

Same as Find() here we can pass filter,options with filter we should filter to specific filed where we wanna chage most of the time we filter on unique field like \_id which is for each elemnt is unique also we can pass update fileds
`db.collection.updateOne(filter, update, options)`

The update is a document that specifies the change to apply.

#### $set operator

The (set) operator allows you to replace the value of a field with a specified value. The $set operator has the following syntax:

**If the field doesn’t exist, the $set operator will add the new field with the specified value to the document as long as the new field doesn’t violate a type constraint.**

```go
func main() {

	result, err := findOneAndUpdate()

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*result)
	// result is : {xTablet 345345}

}

func findOneAndUpdate() (*Product, error) {
	id, err := primitive.ObjectIDFromHex("6459b6400f70b1a5cd54c013")
	if err != nil {
		return nil, err
	}

	filter := bson.M{"_id": id}

	update := bson.M{"$set": bson.M{"price": 345345}}

	projection := bson.M{"_id": 0, "price": 1, "name": 1}

	opts := options.FindOneAndUpdate().SetReturnDocument(options.After).SetProjection(projection)

	var updatedProduct Product
	err = prodCollection.FindOneAndUpdate(ctx, filter, update, opts).Decode(&updatedProduct)
	if err != nil {
		return nil, err
	}

	return &updatedProduct, nil
}
```

---

## updateMany: Update Multiple Documents

The updateMany() method allows you to update all documents that satisfy a condition.

The following shows the syntax of the updateMany() method:

`db.collection.updateMany(filter, update, options)`

The query returns the following result:

` MatchedCount: ,
  ModifiedCount: ,
  UpsertedCount: ,
  UpsertedID:`

```go
func main() {

	err := updateMany()
	if err != nil {
		log.Fatal(err)
	}

	//fmt.Println(*result)
	// result is : {xTablet 345345}

}

func updateMany() error {
	filter := bson.M{"price": bson.M{"$gt": 800}}

	update := bson.M{"$set": bson.M{"price": 990}}

	opts := options.Update()

	updatedProduct, err := prodCollection.UpdateMany(ctx, filter, update, opts)
	if err != nil {
		return nil
	}

	fmt.Println("MatchedCount", updatedProduct.MatchedCount)
	fmt.Println("ModifiedCount", updatedProduct.ModifiedCount)
	fmt.Println("UpsertedCount", updatedProduct.UpsertedCount)
	fmt.Println("UpsertedID", updatedProduct.UpsertedID)

	return nil
}
```

---

## $inc: Increase / Decrease Field Value

Sometimes, you need to increment the value of one or more fields by a specified value. In this case, you can use the update() method with the $inc operator.

`{ $inc: {<field1>: <amount1>, <field2>: <amount2>, ...} }`

if the field doesn’t exist, the $inc creates the field and sets the field to the specified amount.

also we can decreased the fields using $inc

```go
func main() {

	updatedProduct, err := inc()
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*updatedProduct)
	// each time where we run program the value is increased
	// result is : {xPhone 1040} run again : {xPhone 1090}
}

func inc() (*Product, error) {

	id, err := primitive.ObjectIDFromHex("6459b6400f70b1a5cd54c012")
	if err != nil {
		return nil, err
	}

	filter := bson.M{"_id": id}

	update := bson.M{"$inc": bson.M{"price": 50}}

	projection := bson.M{"_id": 0, "price": 1, "name": 1}

	opts := options.FindOneAndUpdate().SetReturnDocument(options.After).SetProjection(projection)

	var updatedProduct Product
	err = prodCollection.FindOneAndUpdate(ctx, filter, update, opts).Decode(&updatedProduct)
	if err != nil {
		return nil, err
	}

	return &updatedProduct, nil
}
```

---

## $min: Update Field Value

The $min operator is a field update operator that allows you to update the value of a field to a specified value if the specified value is less than (lt) the current value of the field.

```go
func main() {

	updatedProduct, err := min()
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*updatedProduct)
	// result is : {xPhone 500}
	// updated because the price is higher than 500
	// if run again program it dosn't work because now the price isnt higher
	// than 500
}

func min() (*Product, error) {

	id, err := primitive.ObjectIDFromHex("6459b6400f70b1a5cd54c012")
	if err != nil {
		return nil, err
	}

	filter := bson.M{"_id": id}

	update := bson.M{"$min": bson.M{"price": 500}}

	projection := bson.M{"_id": 0}

	opts := options.FindOneAndUpdate().SetReturnDocument(options.After).SetProjection(projection)

	var updatedProduct Product
	err = prodCollection.FindOneAndUpdate(ctx, filter, update, opts).Decode(&updatedProduct)
	if err != nil {
		return nil, err
	}

	return &updatedProduct, nil
}
```

---

## $max: Update Field Value

#### the exact same like min but update value if that value not higher than previous value

---

## $mul: Mutiply Field By a Number

The $mul is a field update operator that allows you to multiply the value of a field by a specified number.

The field that you want to update must contain a numeric value

```go
func main() {

	updatedProduct, err := mul()
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*updatedProduct)
	// previous number : {xPhone 280000}
	// after multiply query result is : {xPhone 196000000}
}

func mul() (*Product, error) {

	id, err := primitive.ObjectIDFromHex("6459b6400f70b1a5cd54c012")
	if err != nil {
		return nil, err
	}

	filter := bson.M{"_id": id}

	update := bson.M{"$mul": bson.M{"price": 700}}

	projection := bson.M{"_id": 0}

	opts := options.FindOneAndUpdate().SetReturnDocument(options.After).SetProjection(projection)

	var updatedProduct Product
	err = prodCollection.FindOneAndUpdate(ctx, filter, update, opts).Decode(&updatedProduct)
	if err != nil {
		return nil, err
	}

	return &updatedProduct, nil
}
```

---

## $unset: Remove Fields

Sometimes, you may want to remove one or more fields from a document. In order to do it, you can use the $unset operator.

The $unset is a field update operator that completely removes a particular field from a document.

`{ $unset: {<field>: "", ... }}`

```go
func main() {

	updatedProduct, err := unset()
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*updatedProduct)
	// result is : { 196000000}
	// name deleted
}

func unset() (*Product, error) {

	id, err := primitive.ObjectIDFromHex("6459b6400f70b1a5cd54c012")
	if err != nil {
		return nil, err
	}

	filter := bson.M{"_id": id}

	update := bson.M{"$unset": bson.M{"name": ""}}

	projection := bson.M{"_id": 0}

	opts := options.FindOneAndUpdate().SetReturnDocument(options.After).SetProjection(projection)

	var updatedProduct Product
	err = prodCollection.FindOneAndUpdate(ctx, filter, update, opts).Decode(&updatedProduct)
	if err != nil {
		return nil, err
	}

	return &updatedProduct, nil
}
```

---

## $rename: Rename Fields

Sometimes, you want to rename a field in a document e.g., when it is misspelled or not descriptive enough. In this case, you can use the $rename operator.

The $rename is a field update operator that allows you to rename a field in a document to the new one.

`{ $rename: { <field_name>: <new_field_name>, ...}}`

```go
func main() {

	updatedProduct, err := rename()
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*updatedProduct)
	// result is : { 990} name changed to fName
}

func rename() (*Product, error) {

	id, err := primitive.ObjectIDFromHex("6459b6400f70b1a5cd54c013")
	if err != nil {
		return nil, err
	}

	filter := bson.M{"_id": id}

	update := bson.M{"$rename": bson.M{"name": "fName"}}

	projection := bson.M{"_id": 0}

	opts := options.FindOneAndUpdate().SetReturnDocument(options.After).SetProjection(projection)

	var updatedProduct Product
	err = prodCollection.FindOneAndUpdate(ctx, filter, update, opts).Decode(&updatedProduct)
	if err != nil {
		return nil, err
	}

	return &updatedProduct, nil
}
```

---

## deleteOne

The deleteOne() method allows you to delete a single document from a collection.

`db.collection.deleteOne(filter, option)`

The deleteOne() method accepts two arguments:

- filter

- option

deleteOne just return acknowledge of deleting instead we use FindOneAndDelete() to get data also

```go
func main() {

	updatedProduct, err := deleteOne()
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(*updatedProduct)
	// result is : {SmartTablet 990}
	// which is deleted from db
}

func deleteOne() (*Product, error) {
	id, err := primitive.ObjectIDFromHex("6459b6400f70b1a5cd54c014")
	if err != nil {
		return nil, err
	}

	filter := bson.M{"_id": id}

	projection := bson.M{"_id": 0}

	opts := options.FindOneAndDelete().SetProjection(projection)

	var deletedProduct Product
	err = prodCollection.FindOneAndDelete(ctx, filter, opts).Decode(&deletedProduct)
	if err != nil {
		return nil, err
	}

	return &deletedProduct, nil
}
```

---

## deleteMany

The deleteMany() method allows you to remove all documents that match a condition from a collection.

`db.collection.deleteMany(filter, option)`

The deleteMany() returns a document containing the deleteCount field that stores the number of deleted documents.

To delete a single document from a collection, you can use the deleteOne() method.

```go
func main() {

	err := deleteMany()
	if err != nil {
		log.Fatal(err)
	}

	// all the products has been deleted

}

func deleteMany() error {

	filter := bson.M{"price": bson.M{"$gt": 500}}

	result, err := prodCollection.DeleteMany(ctx, filter)
	if err != nil {
		return nil
	}

	fmt.Println(result.DeletedCount)
	// return three becuase all products are price its bigger than 500

	return nil
}
```

---

## Aggregation Pipeline

MongoDB aggregation operations allow you to process multiple documents and return the calculated results.

Typically, you use **aggregation operations to group documents by specific field values** and perform aggregations on the grouped documents to return computed results.

For example, you can use **aggregation operations to take a list of sales** orders and **calculate the total sales** amounts grouped by the products.

To perform aggregation operations, you use aggregation pipelines. An **aggregation pipeline contains one or more stages** that process the input documents:

Each stage in the aggregation pipeline performs an operation on the input documents and **returns the output** documents. **The output documents are then passed to the next stage**. The final stage returns the calculated result.

The operations on each stage can be one of the following:

- $project – select fields for the output documents.

- $match – select documents to be processed.

- $limit – limit the number of documents to be passed to the next stage.

- $skip – skip a specified number of documents.

- $sort – sort documents.

- $group – group documents by a specified key.

and so on ...

`db.collection.aggregate([{ $match:...},{$group:...},{$sort:...}]);`

In this example we calculate all price of products and resturn

```go
func main() {

	totalPrice, err := getTotalPrice()

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(totalPrice)

}

func getTotalPrice() (float64, error) {

	pipeline := []bson.M{
		{"$group": bson.M{
			"_id": nil,
			"totalPrice": bson.M{
				"$sum": "$price",
			},
		}},
	}

	cursor, err := prodCollection.Aggregate(ctx, pipeline)
	if err != nil {
		return 0, err
	}
	defer func() {
		if cerr := cursor.Close(ctx); cerr != nil {
			err = cerr
		}
	}()

	var result struct {
		TotalPrice float64 `bson:"totalPrice"`
	}

	if cursor.Next(ctx) {
		if err := cursor.Decode(&result); err != nil {
			return 0, err
		}
	}

	return result.TotalPrice, nil
}
```

---

## $avg

The MongoDB avg returns the average value of numeric values. The syntax of the $avg is as follows:

**The $avg ignores the non-numeric and missing values. If all values are non-numeric, the $avg returns null.**

```go
func main() {

	averagePrice, err := getAveragePrice()

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(averagePrice)

}

func getAveragePrice() (float64, error) {

	pipeline := []bson.M{
		{"$group": bson.M{
			"_id": nil,
			"averagePrice": bson.M{
				"$avg": "$price",
			},
		}},
	}

	cursor, err := prodCollection.Aggregate(ctx, pipeline)
	if err != nil {
		return 0, err
	}
	defer func() {
		if cerr := cursor.Close(ctx); cerr != nil {
			err = cerr
		}
	}()

	var result struct {
		AveragePrice float64 `bson:"averagePrice"`
	}

	if cursor.Next(ctx) {
		if err := cursor.Decode(&result); err != nil {
			return 0, err
		}
	}

	return result.AveragePrice, nil
}
```

---

## $count

MongoDB $count returns the number of documents in a group. Here’s the syntax of the

Note that the $count does not accept any parameters.

The $count is functionally equivalent to using the following $sum in the $group stage:

`{ $sum: 1 }`

```go

func main() {

	totalPrice, err := count()

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(totalPrice)

}

func count() (float64, error) {

	pipeline := []bson.M{
		{"$group": bson.M{
			"_id": nil,
			"countDocs": bson.M{
				"$count": bson.M{},
			},
		}},
	}

	cursor, err := prodCollection.Aggregate(ctx, pipeline)
	if err != nil {
		return 0, err
	}
	defer func() {
		if cerr := cursor.Close(ctx); cerr != nil {
			err = cerr
		}
	}()

	var result struct {
		CountDocs float64 `bson:"countDocs"`
	}

	if cursor.Next(ctx) {
		if err := cursor.Decode(&result); err != nil {
			return 0, err
		}
	}

	return result.CountDocs, nil
}
```

---

## $max

The MongoDB (max) returns the maximum value. The $max operator uses both value and type for comparing according to the BSON

If you apply the (max) to the field that has a null or missing value in all documents, the $max returns null.

However, if you apply the (max) to the field that has a null or missing value in some documents, but not all, the (max) only considers non-null and non-missing values for that field.

```go
func main() {

	totalPrice, err := aggregateMax()

	if err != nil {
		log.Fatal(err)
	}

	// max price in all documents
	fmt.Println(totalPrice)

}

func aggregateMax() (float64, error) {

	pipeline := []bson.M{
		{"$group": bson.M{
			"_id": nil,
			"maxPrice": bson.M{
				"$max": "$price",
			},
		}},
	}

	cursor, err := prodCollection.Aggregate(ctx, pipeline)
	if err != nil {
		return 0, err
	}
	defer func() {
		if cerr := cursor.Close(ctx); cerr != nil {
			err = cerr
		}
	}()

	var result struct {
		MaxPrice float64 `bson:"maxPrice"`
	}

	if cursor.Next(ctx) {
		if err := cursor.Decode(&result); err != nil {
			return 0, err
		}
	}

	return result.MaxPrice, nil
}
```

---

## $min

#### Same as $max, but returns minimum value in the group

---

## MongoDB Indexes

---

## Create Index

---

## Unique Index

---

## Compound index

---

## Drop Index

---
