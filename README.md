//  ----------------------------- app  ------------------------------
package app;
 
import java.sql.Date;
import java.util.Arrays;

import com.mongodb.client.MongoClient;

import dao.CustomerDao;
import dao.ProductDao;
import entity.Address;
import entity.Customer;
import entity.Phone;
import utils.DBConnection;

public class App {
	private static final String DB_NAME = "BikeStores";

	public static void main(String[] args) {
		
		MongoClient mongoClient = DBConnection.getInstance().getMongoClient();
		
//		ProductDao productDao = new ProductDao(mongoClient, DB_NAME);
		CustomerDao customerDao = new CustomerDao(mongoClient, DB_NAME);
		
		Address address = new Address("London", "Ohi you", "Backing", 19750);
//		
		Phone phone = new Phone("0906461526", "Jorker");
		
		Customer customer = new Customer("113913", "trung", "hoai", address, new Date(0), "hoaiiuh@gmail.com" );
		
		customer.setPhones(Arrays.asList(phone));
		
//		System.out.println(customerDao.addCustomer(customer));
		
		System.out.println(customerDao.updateCustomer("113912", customer));
//		
		System.out.println(customerDao.getCustomer("113912"));
		
//		System.out.println(productDao.getProduct(1));
		
//		System.out.println(customerDao.getCustomer("CHAR5"));
	}
}
--------------------------------- dao ------------------------------------------------
package dao;

import com.mongodb.MongoClientSettings;
import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.model.Filters;
import com.mongodb.client.model.Updates;
import com.mongodb.client.result.InsertOneResult;

import org.bson.codecs.configuration.CodecRegistries;
import org.bson.codecs.configuration.CodecRegistry;
import org.bson.codecs.pojo.PojoCodecProvider;
import org.bson.codecs.pojo.annotations.BsonProperty;

import entity.Customer;
import entity.Product;

public class CustomerDao {
	
	private MongoCollection<Customer> proCol;
	
	public CustomerDao(MongoClient mongoClient, String dbName) {
		PojoCodecProvider pojoCodecProvider = PojoCodecProvider.builder().automatic(true).build();
		CodecRegistry codecRegistry = CodecRegistries.fromRegistries(MongoClientSettings.getDefaultCodecRegistry(),
				CodecRegistries.fromProviders(pojoCodecProvider));
		proCol = mongoClient.getDatabase(dbName).getCollection("customers", Customer.class)
				.withCodecRegistry(codecRegistry);
	}
	
	public Customer getCustomer(String id) {
		return proCol.find(Filters.eq("_id", id)).first();
	}

	public boolean addCustomer(Customer customer) {
		InsertOneResult result = proCol.insertOne(customer);
		if (result != null) {
			return true;
		}
		return false;
	}
	
	public boolean updateCustomer(String id, Customer customer) {
		try {
			Object result = proCol.updateOne(Filters.eq("_id", id), Updates.combine(
				Updates.set("first_name", customer.getFirstName()),
				Updates.set("last_name", customer.getLastName()),
				Updates.set("registration_date", customer.getRegistrationDate()),
				Updates.set("email", customer.getEmail()),
				Updates.set("phones", customer.getPhones()),
				Updates.set("address.city", customer.getAddress().getCity()),
				Updates.set("address.state", customer.getAddress().getState()),
				Updates.set("address.street", customer.getAddress().getStreet()),
				Updates.set("address.zop_code", customer.getAddress().getZipCode())
			));
			System.out.println(result);
			return true;
		} catch (Exception e) {
			System.out.println("khong update duoc");
		}
		return false;
	}
}





package dao;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import com.mongodb.MongoClientSettings;
import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.model.Filters;
import com.mongodb.client.model.Updates;
import com.mongodb.client.result.DeleteResult;
import com.mongodb.client.result.InsertManyResult;
import com.mongodb.client.result.InsertOneResult;

import org.bson.Document;
import org.bson.codecs.configuration.CodecRegistries;
import org.bson.codecs.configuration.CodecRegistry;
import org.bson.codecs.pojo.PojoCodecProvider;
import org.bson.conversions.Bson;

import entity.Product;

public class ProductDao {
	
	private MongoCollection<Product> proCol;
	
	public ProductDao(MongoClient mongoClient, String dbName) {
		PojoCodecProvider pojoCodecProvider = PojoCodecProvider.builder().automatic(true).build();
		CodecRegistry codecRegistry = CodecRegistries.fromRegistries(MongoClientSettings.getDefaultCodecRegistry(),
				CodecRegistries.fromProviders(pojoCodecProvider));
		proCol = mongoClient.getDatabase(dbName).getCollection("products", Product.class)
				.withCodecRegistry(codecRegistry);
	}
	// x
	public Product getProduct(long id) {
		return proCol.find(Filters.eq("_id", id)).first();
	}
	public List<Product> getProducts(int limit) {
		return proCol.find().limit(limit).into(new ArrayList<Product>());
	}

	// them nhieu san pham x
	public int addProducts(List<Product> products) {
		InsertManyResult result = proCol.insertMany(products);
		return result.getInsertedIds().size();
	}
	// x
	public DeleteResult deleteZip(long id) {
		Bson query = Filters.eq("_id",id);
		return proCol.deleteOne(query);
	}
	
	// them 1 san pham x
	public boolean addProduct(Product product) {
		InsertOneResult result = proCol.insertOne(product);
		if (result != null) {
			return true;
		}
		return false;
	}

	// cap nhat mot san pham theo id
	public boolean updateProduct(long id, Product product) {
		try {
			Object result = proCol.updateOne(Filters.eq("_id", id), Updates.set("brand_name", product.getBrand()));
//			Object result1 = proCol.updateOne(Filters.eq("_id", id),
//					Updates.set("category_name", product.getCategory()));
			System.out.println(result);
//			System.out.println(result1);
			return true;
		} catch (Exception e) {
			System.out.println("khong update duoc");
		}
		return false;
	}

	public boolean updateProduct1(long id, Product product) {
		try {
			Object result = proCol.updateOne(Filters.eq("_id", id), Updates.combine(
				Updates.set("brand_name", product.getBrand()),
				Updates.set("price", product.getPrice()),
				Updates.set("category_name", product.getCategory()),
				Updates.set("model_year", product.getModelYear()),
				Updates.set("colors", product.getColors()))
			);
			System.out.println(result);
			return true;
		} catch (Exception e) {
			System.out.println("khong update duoc");
		}
		return false;
	}

	public boolean updateProduct2(long id, Product product) {
		try {
			Object result = proCol.updateMany(Filters.gt("_id", id), Updates.combine(
					Updates.set("brand_name", product.getBrand()), Updates.set("price", product.getPrice()),
					Updates.set("category_name", product.getCategory()),
					Updates.set("model_year", product.getModelYear()), Updates.set("colors", product.getColors())));
			System.out.println(result);
			return true;
		} catch (Exception e) {
			System.out.println("khong update duoc");
		}
		return false;
	}

	// cap nhat nhieu san pham
	public boolean updateManyProduct(double price, Product product) {
		try {
			Object result = proCol.updateMany(Filters.gte("price", price),
					Updates.set("brand_name", product.getBrand()));
			System.out.println(result);
			return true;
		} catch (Exception e) {
			System.out.println("khong update duoc");
		}
		return false;
	}

//	db.products.aggregate([{$group:{_id:null, products:{$push:"$$ROOT"},max:{$max:'$price'}}},
//	{$unwind:'$products'},
//	{$match:{$expr:{$eq:["$max","$products.price"]}}},
//	{$replaceWith:'$products'}]).pretty()
	// lay san pham co gia cao nhat
	public List<Product> getProductsMax() {
		return proCol.aggregate(
				Arrays.asList(Document.parse("{$group:{_id:null, products:{$push:\"$$ROOT\"},max:{$max:'$price'}}}"),
						Document.parse("{$unwind:'$products'}"),
						Document.parse("{$match:{$expr:{$eq:[\"$max\",\"$products.price\"]}}}"),
						Document.parse("{$replaceWith:'$products'}")))
				.into(new ArrayList<>());
	}

//	db.products.aggregate([{$lookup:{from:'orders', localField:'_id', foreignField:'order_details.product_id', as:'rs'}},
//	{$match:{rs:{$size:0}}},
//	{$unset:'rs'}]).pretty()
	// lay nhung sp ban it nhat (o ban dc cai nao)
	public List<Product> getProductsBad() {
		return proCol.aggregate(Arrays.asList(Document
				.parse("{$lookup:{from:'orders', localField:'_id', foreignField:'order_details.product_id', as:'rs'}}"),
				Document.parse("{$match:{rs:{$size:0}}}"), Document.parse("{$unset:'rs'}"))).into(new ArrayList<>());
	}
}

------------------------ entity --------------- 
 *  (C) Copyright 2020 IUH. All rights reserved.
 * 
 *  @author: VinhHien
 *  @date: Oct 21, 2020
 *  @version: 1.0
 */

package entity;

import java.util.Date;
import java.util.List;

import org.bson.codecs.pojo.annotations.BsonId;
import org.bson.codecs.pojo.annotations.BsonProperty;

public class Customer {
	@BsonId
	private String customerId;
	@BsonProperty("first_name")
	private String firstName;
	@BsonProperty("last_name")
	private String lastName;
	private Address address;
	@BsonProperty("registration_date")
	private Date registrationDate;
	private String email;
	private List<Phone> phones;

	public Customer() {
	}
	
	/**
	 * @param _id
	 * @param firstName
	 * @param lastName
	 * @param address
	 * @param registrationDate
	 * @param email
	 */
	public Customer(String customerId, String firstName, String lastName, Address address, Date registrationDate,
			String email) {
		this.customerId = customerId;
		this.firstName = firstName;
		this.lastName = lastName;
		this.address = address;
		this.registrationDate = registrationDate;
		this.email = email;
	}


	/**
	 * @return the customerId
	 */
	public String getCustomerId() {
		return customerId;
	}

	/**
	 * @param customerId the customerId to set
	 */
	public void setCustomerId(String customerId) {
		this.customerId = customerId;
	}

	/**
	 * @return the firstName
	 */
	public String getFirstName() {
		return firstName;
	}

	/**
	 * @param firstName the firstName to set
	 */
	public void setFirstName(String firstName) {
		this.firstName = firstName;
	}

	/**
	 * @return the lastName
	 */
	public String getLastName() {
		return lastName;
	}

	/**
	 * @param lastName the lastName to set
	 */
	public void setLastName(String lastName) {
		this.lastName = lastName;
	}

	/**
	 * @return the address
	 */
	public Address getAddress() {
		return address;
	}

	/**
	 * @param address the address to set
	 */
	public void setAddress(Address address) {
		this.address = address;
	}

	/**
	 * @return the registrationDate
	 */
	public Date getRegistrationDate() {
		return registrationDate;
	}

	/**
	 * @param registrationDate the registrationDate to set
	 */
	public void setRegistrationDate(Date registrationDate) {
		this.registrationDate = registrationDate;
	}

	/**
	 * @return the email
	 */
	public String getEmail() {
		return email;
	}

	/**
	 * @param email the email to set
	 */
	public void setEmail(String email) {
		this.email = email;
	}


	@Override
	public String toString() {
		return "Customer [_id=" + customerId + ", firstName=" + firstName + ", lastName=" + lastName + ", address=" + address
				+ ", registrationDate=" + registrationDate + ", email=" + email + phones + "]";
	}

	public List<Phone> getPhones() {
		return phones;
		
	}

	public void setPhones(List<Phone> phones) {
		this.phones = phones;
		
	}
	
}



 *  (C) Copyright 2020 IUH. All rights reserved.
 * 
 *  @author: VinhHien
 *  @date: Oct 21, 2020
 *  @version: 1.0
 */

package entity;

public class Phone {
	private String number;
	private String type;
	
	public Phone() {
	}

	/**
	 * @param number
	 * @param type
	 */
	public Phone(String number, String type) {
		this.number = number;
		this.type = type;
	}

	/**
	 * @return the number
	 */
	public String getNumber() {
		return number;
	}

	/**
	 * @param number the number to set
	 */
	public void setNumber(String number) {
		this.number = number;
	}

	/**
	 * @return the type
	 */
	public String getType() {
		return type;
	}

	/**
	 * @param type the type to set
	 */
	public void setType(String type) {
		this.type = type;
	}

	@Override
	public String toString() {
		return "Phone [number=" + number + ", type=" + type + "]";
	}
	
}




 -------------------------------- db connect ------------------------------
 package utils;
 
import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoClients; 

public class DBConnection {
	private static DBConnection instance = null;
	private MongoClient mongoClient;
	
	private DBConnection() { 
		mongoClient = MongoClients.create(); // neu connect den local thi bo doi so clientSettings la dc
	}
	
	public synchronized static DBConnection getInstance() {
		if(instance == null)
			instance = new DBConnection();
		return instance;
	}
	public MongoClient getMongoClient() {
		return mongoClient;
	}
}


--------------------------------- pom --------------------
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>mongodb</groupId>
  <artifactId>mongo-pojo-gk</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <dependencies>
  		<dependency>
			<groupId>org.mongodb</groupId>
			<artifactId>mongodb-driver-sync</artifactId>
			<version>4.7.1</version>
		</dependency>
		<dependency>
			<groupId>org.apache.logging.log4j</groupId>
			<artifactId>log4j-slf4j-impl</artifactId>
			<version>2.17.1</version>
		</dependency>
  </dependencies>
</project>
--------------------------------------------------- mapper-----------------------
package bus.impl;

import org.bson.Document;

import entity.Location;
import entity.Zip;

public class Mapper {

	public static Zip fromZip(Document doc) { //decode
		
		Document locDoc = (Document) doc.get("loc");
//		Location loc = new Location(locDoc.getDouble("y"), locDoc.getDouble("x"));	
		
		return new Zip(doc.getObjectId("_id").toHexString(), 
				doc.getString("city"), 
				doc.getString("zip"), 
				new Location(locDoc.getDouble("y"), locDoc.getDouble("x")),
				doc.getInteger("pop"), 
				doc.getString("state"));
	}
}

--------------------------------------- service impl -----------------------------------
package bus.impl;

import java.util.List;
import java.util.stream.Collectors;

import org.bson.Document;

import bus.ZipService;
import dao.ZipDao;
import entity.Zip;

public class ZipServiceImpl implements ZipService{
	
	private ZipDao zipDao;
	
	public ZipServiceImpl() {
		zipDao = new ZipDao();
	}

	@Override
	public List<Zip> getZipByCity(String city) {
		if(city != null && !city.trim().equals("")) {
			List<Document> docs = zipDao.getZipByCity(city);
			return docs.stream().map(doc -> {
				return Mapper.fromZip(doc);
			}).collect(Collectors.toList());
		}
		return null;
	}

	@Override
	public Zip getZip(String id) {
		
//		if(id != null && !id.trim().equals("") && ObjectId.isValid(id)) {
			Document doc = zipDao.getZip(id);
			Zip zip = Mapper.fromZip(doc);
			return zip;
//		}
//		return null;
	}

	@Override
	public int getPopByCity(String city) {
		if(city != null && !city.equals(""))
			return zipDao.getPopByCity(city);
		return 0;
	}
}
------------------------------------------------ mongo client ------------------------------------------
package util;

import java.util.concurrent.TimeUnit;

import com.mongodb.ConnectionString;
import com.mongodb.MongoClientSettings;
import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoClients;

public class AtlasMongoClient {
	private static AtlasMongoClient instane = null;
	private MongoClient mongoClient;
	
	private AtlasMongoClient() {
		String uri = "mongodb+srv://hoai19511341:1234@hoaicluster.1n1ovus.mongodb.net/?retryWrites=true&w=majority";
		ConnectionString connectionString = new ConnectionString(uri);
		MongoClientSettings clientSettings = 
				MongoClientSettings
				.builder()
				.applicationName("Exercise")
				.applyConnectionString(connectionString)
				.applyToConnectionPoolSettings(builder -> builder.maxSize(0))
				.applyToSocketSettings(builder -> builder.connectTimeout(5, TimeUnit.SECONDS))
				.build();
		mongoClient = MongoClients.create(clientSettings); // neu connect den local thi bo doi so clientSettings la dc
	}
	
	public static synchronized AtlasMongoClient getInstane() {
		if(instane == null)
			instane = new AtlasMongoClient();
		return instane;
	}
	public MongoClient getMongoClient() {
		return mongoClient;
	}
}


-------------------------------- mapper dao -----------------
package dao;

import static com.mongodb.client.model.Filters.eq;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import org.bson.BsonNull;
import org.bson.Document;
import org.bson.conversions.Bson;
import org.bson.types.ObjectId;

import com.mongodb.client.AggregateIterable;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.model.Accumulators;
import com.mongodb.client.model.Aggregates;
import com.mongodb.client.model.Filters;
import com.mongodb.client.model.Projections;

import util.AtlasMongoClient;

public class ZipDao {
	
	private MongoCollection<Document> zipCol;

	public ZipDao() {
		zipCol = AtlasMongoClient.getInstane().getMongoClient().getDatabase("sample_training").getCollection("zips");
	}

	// Tìm các document có city là PALMER
	public List<Document> getZipByCity(String city) {
		return zipCol.find(eq("city", city)).into(new ArrayList<>());
	}

	public Document getZip(String id) {

		return zipCol.find(Filters.eq("_id", new ObjectId(id))).first();
	}

	// Tìm các document có dân số >100000
	// db.zips.find({pop:{$gt:100000}})
	public List<Document> getZipByPop(int pop) {

		Document filter = Document.parse("{pop:{$gt:" + pop + "}}");

		// Document filter = new Document()
		// .append("pop", new Document().append("$gt", pop));

		return zipCol.find(filter).into(new ArrayList<>());

		// return zipCol
		// .find(Filters.gt("pop", pop))
		// .into(new ArrayList<>());
	}

	// db.zips.aggregate([{$match:{"city" : "CRANE HILL"}},{$project:{pop:1,
	// _id:0}}])
	// Tìm dân số của thành phố FISHERS ISLAND

	public int getPopByCity(String city) {
		Document match = Document.parse("{$match:{\"city\" : \"" + city + "\"}}"); // {$match:{"city": "city"}}
		Document project = Document.parse("{$project:{pop:1, _id:0}}");
		Document firstDoc = zipCol.aggregate(Arrays.asList(match, project)).first();
//		System.out.println(Arrays.asList(match, project ));
//		return 1;
		return firstDoc.getInteger("pop");
	}

//	db.zips.aggregate([{$match:{"state" : "NY"}},{$group:{_id:null, total:{$sum:'$pop'}}},{$project:{_id:0}}])
	public int getPopByState(String state) {
		Bson match = Aggregates.match(Filters.eq("state", state));
		Bson group = Aggregates.group(Filters.eq("_id", null), Accumulators.sum("total", "$pop"));
		Bson project = Aggregates.project(Projections.include("total"));
		Document aggregate = zipCol.aggregate(Arrays.asList(match, group, project)).first();
//		System.out.println(Arrays.asList(match, group, project));
		return aggregate.getInteger("total");
	}

	public int getPopByState2(String state) { // tu lam
		Document match = Document.parse("{$match:{state : \"" + state + "\"}}"); // {$match:{"city": "city"}}
		Document group = Document.parse("{$group:{_id: null, total:{$sum: \"$pop\"}}}");
//		Document project = Document.parse("{$project:{_id: 0, total: 1}");
		Document firstDoc = zipCol.aggregate(Arrays.asList(match, group)).first();
		System.out.println(firstDoc);
//		System.out.println(Arrays.asList(match, group, project ));
//		return 1;
		return firstDoc.getInteger("total");
	}

	public int getPopByState3(String state) { // tu lam

//		Document match = new Document("$match", new Document("name", "Lisa Rasmussen"));
//		Document count = new Document("$count", "n");

		Document match = new Document("$match", new Document("state", state));
		Document group = new Document("$group",
				Arrays.asList(new Document("_id", null), new Document("$total", new Document("$sum", "$pop"))));
		Document project = new Document("$project", Arrays.asList(new Document("_id", 0), new Document("total", 1)));
		Document firstDoc = zipCol.aggregate(Arrays.asList(match, group, project)).first();

//		Document match = Document.parse("{$match:{state : \""+state+"\"}}"); //{$match:{"city": "city"}}
//		Document group = Document.parse("{$group:{_id: null, total:{$sum: \"$pop\"}}}");
//		Document project = Document.parse("{$project:{_id: 0, total: 1}");
//		Document firstDoc = zipCol.aggregate(Arrays.asList(match, group, project )).first();
		
		System.out.println(firstDoc);
//		System.out.println(Arrays.asList(match, group, project ));
		
//		return 1;
		return firstDoc.getInteger("total");
	}
}

