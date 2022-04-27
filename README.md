# RestApiTestExample

## 1. Create constants for your api urls

Create a folder constants and put inside the url and the endpoints that u will use (for this example i'm using the public appi on https://jsonplaceholder.typicode.com):

    class ApiConstants {
      static String baseUrl = "https://jsonplaceholder.typicode.com";
      static String userEndpoint = "/users";
    }


## 2. Create a modal class

You will have to make a modal class for future use of the Json/XML... data that you will recibe, the class should implements a pair of methods to parse the information type. You can do this by hand, or using some external resource like: https://app.quicktype.io

In this example I created a UserModel class:

    import 'dart:convert';
    
    List<UserModel> userModelFromJson(String str) =>
        List<UserModel>.from(json.decode(str).map((x) => UserModel.fromJson(x)));
    
    String userModelToJson(List<UserModel> data) =>
        json.encode(List<dynamic>.from(data.map((x) => x.toJson())));
    
    class UserModel {
      UserModel({
        required this.id,
        required this.name,
        required this.username,
        required this.email,
        required this.address,
        required this.phone,
        required this.website,
        required this.company,
      });
    
      int id;
      String name;
      String username;
      String email;
      Address address;
      String phone;
      String website;
      Company company;
    
      factory UserModel.fromJson(Map<String, dynamic> json) => UserModel(
            id: json["id"],
            name: json["name"],
            username: json["username"],
            email: json["email"],
            address: Address.fromJson(json["address"]),
            phone: json["phone"],
            website: json["website"],
            company: Company.fromJson(json["company"]),
          );
    
      Map<String, dynamic> toJson() => {
            "id": id,
            "name": name,
            "username": username,
            "email": email,
            "address": address.toJson(),
            "phone": phone,
            "website": website,
            "company": company.toJson(),
          };
    }
    
    class Address {
      Address({
        required this.street,
        required this.suite,
        required this.city,
        required this.zipcode,
        required this.geo,
      });
    
      String street;
      String suite;
      String city;
      String zipcode;
      Geo geo;
    
      factory Address.fromJson(Map<String, dynamic> json) => Address(
            street: json["street"],
            suite: json["suite"],
            city: json["city"],
            zipcode: json["zipcode"],
            geo: Geo.fromJson(json["geo"]),
          );
    
      Map<String, dynamic> toJson() => {
            "street": street,
            "suite": suite,
            "city": city,
            "zipcode": zipcode,
            "geo": geo.toJson(),
          };
    }
    
    class Geo {
      Geo({
        required this.lat,
        required this.lng,
      });
    
      String lat;
      String lng;
    
      factory Geo.fromJson(Map<String, dynamic> json) => Geo(
            lat: json["lat"],
            lng: json["lng"],
          );
    
      Map<String, dynamic> toJson() => {
            "lat": lat,
            "lng": lng,
          };
    }
    
    class Company {
      Company({
        required this.name,
        required this.catchPhrase,
        required this.bs,
      });
    
      String name;
      String catchPhrase;
      String bs;
    
      factory Company.fromJson(Map<String, dynamic> json) => Company(
            name: json["name"],
            catchPhrase: json["catchPhrase"],
            bs: json["bs"],
          );
    
      Map<String, dynamic> toJson() => {
            "name": name,
            "catchPhrase": catchPhrase,
            "bs": bs,
          };
    }
    

## 3. Create the Api call methods

At this point, we need to create the call methods to the api, if we get a 200 "ok" response, then we will call the function to model the Json data to our model class:

    import 'dart:developer';
    import 'package:http/http.dart' as http;
    import 'package:restapitest/constants/constants.dart';
    import 'package:restapitest/model/user_model.dart';
    
    class ApiService {
      Future<List<UserModel>?> getUsers() async {
        try {
          var url = Uri.parse(ApiConstants.baseUrl + ApiConstants.userEndpoint);
          var response = await http.get(url);
          if (response.statusCode == 200) {
            List<UserModel> _model = userModelFromJson(response.body);
            return _model;
          }
        } catch (e) {
          log(e.toString());
        }
      }
    }

##4 Use the data

Finally we will use the data on a simple ListView widget. We need to initialize an empty list of our model class:

    class _HomeState extends State<Home> {
      late List<UserModel>? _userModel = [];

then, before our dispose(), we need an aynchronous method to fill our list using the get api method created previously:

    class _HomeState extends State<Home> {
      late List<UserModel>? _userModel = [];
      @override
      void initState() {
        super.initState();
        _getData();
      }
    
      void _getData() async {
        _userModel = (await ApiService().getUsers())!;
        setState(() {
          _userModel;
        });
      }

and last, we create a scaffold with a ListView.builder that get the data as builder parameter:

    import 'package:flutter/material.dart';
    import 'package:restapitest/services/api_services.dart';
    
    import 'model/user_model.dart';
    
    void main() {
      runApp(const MyApp());
    }
    
    class MyApp extends StatelessWidget {
      const MyApp({Key? key}) : super(key: key);
    
      @override
      Widget build(BuildContext context) {
        return MaterialApp(
          title: 'Flutter Demo',
          theme: ThemeData(
            primarySwatch: Colors.blue,
          ),
          home: const Home(title: 'Flutter Demo Home Page'),
        );
      }
    }
    
    class Home extends StatefulWidget {
      const Home({Key? key, required String title}) : super(key: key);
    
      @override
      _HomeState createState() => _HomeState();
    }
    
    class _HomeState extends State<Home> {
      late List<UserModel>? _userModel = [];
      @override
      void initState() {
        super.initState();
        _getData();
      }
    
      void _getData() async {
        _userModel = (await ApiService().getUsers())!;
        setState(() {
          _userModel;
        });
      }
    
      @override
      Widget build(BuildContext context) {
        return Scaffold(
          appBar: AppBar(
            title: const Text('REST API Example'),
          ),
          body: _userModel == null || _userModel!.isEmpty
              ? const Center(
                  child: CircularProgressIndicator(),
                )
              : ListView.builder(
                  itemCount: _userModel!.length,
                  itemBuilder: (context, index) {
                    return Card(
                      child: Column(
                        children: [
                          Row(
                            mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                            children: [
                              Text(_userModel![index].id.toString()),
                              Text(_userModel![index].username),
                            ],
                          ),
                          const SizedBox(
                            height: 20.0,
                          ),
                          Row(
                            mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                            children: [
                              Text(_userModel![index].email),
                              Text(_userModel![index].website),
                            ],
                          ),
                        ],
                      ),
                    );
                  },
                ),
        );
      }
    }
    


## END: That is all
