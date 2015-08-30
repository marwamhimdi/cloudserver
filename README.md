# cloudserver
#!flask/bin/python
from flask import Flask, jsonify, abort, request
from jsonschema import validate
from pymongo import MongoClient
import json
from bson.json_util import dumps
from pprint import pprint
import sys

#Validation schema
SCHEMA = {
    "type" : "object",
    "properties" : {
        "controller" : {
            "type" : "object",
            "properties" : {
                "id" : {"type" : "string"},
                "sensors" : {
                    "type" : "array",
                    "items" : {
                        "type" : "object",
                        "properties" : {
                            "_id" : {"type" : "integer"},
                                "temperature" : {
                                    "type" : "object",
                                    "properties" : {
                                        "value" : {"type" : "number"},
                                        "updateTime" : {"type" : "integer"}
                            },
                                "required" : ["value","updateTime"]
                                },
                                "humidity" : {
                                    "type" : "object",
                                    "properties" : {
                                        "value" : {"type" : "number"},
                                        "updateTime" : {"type" : "integer"}
                            },
                                "required" : ["value","updateTime"]
                                },
                                "luminosity" : {
                                    "type" : "object",
                                    "properties" : {
                                        "value" : {"type" : "number"},
                                        "updateTime" : {"type" : "integer"}
                            },
                                "required" : ["value","updateTime"]
                                },
                                "presence" : {
                                    "type" : "object",
                                    "properties" : {
                                        "value" : {"type" : "boolean"},
                                        "updateTime" : {"type" : "integer"}
                                },
                                "required" : ["value","updateTime"]
                                },
                    },
                    "required" : ["_id","temperature","humidity","luminosity","presence"]
            }
        }
        },
            "required" : ["id","sensors"]
            }
            },
"required" : ["controller"]
}

#API DESCRIPTION
API_DESCRIPTION = """<h3>API DESCRIPTION</h3>
    Route <b>/insertrecord/{$student_name}</b><br/>
    <i>Insert the sensor values in the mongo database.<br/> Method : POST ; Data format : JSON (as shown below) ; $student_name : Your name or id or whatever you want</i><br/><br/>
    Route <b>/getrecords/{$student_name}</b><br/>
    <i>Return all the database records for student $student_name<br/>Data format : JSON - List of records (as shown below)</i><br/><br/>
    JSON format :<br/>
    <b> NB : The server will reject any malformed JSON</b><br/>
    <span style="font-family:'Courier New';font-size:13">{<br/>
    &nbsp;&nbsp;"controller": { <br/>
    &nbsp;&nbsp;&nbsp;&nbsp;"id": "CONTROLLER_ID",<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;"sensors": [ ARRAY OF SENSOR ITEMS ]<br/>
    &nbsp;&nbsp;}<br/>
    }<br/><br/>
    SENSOR ITEM :<br/>
    {<br/>
    &nbsp;&nbsp;"_id": "SENSOR ID"{ <br/>
    &nbsp;&nbsp;"temperature": {<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;"value" : "TEMPERATURE VALUE(number, degree Celsius)",<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;"updateTime" : "TIMESTAMP"<br/>
    &nbsp;&nbsp;},<br/>
    &nbsp;&nbsp;"humidity": {<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;"value" : "HUMIDITY VALUE(number, percentage)",<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;"updateTime" : "TIMESTAMP"<br/>
    &nbsp;&nbsp;},<br/>
    &nbsp;&nbsp;"luminosity": {<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;"value" : "LUMINOSITY VALUE(number, lux)",<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;"updateTime" : "TIMESTAMP"<br/>
    &nbsp;&nbsp;},<br/>
    &nbsp;&nbsp;"presence": {<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;"value" : "PRESENCE VALUE (True/False)",<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;"updateTime" : "TIMESTAMP"<br/>
    &nbsp;&nbsp;}<br/>
    }<br/><br/>
    </span>
    
    """


#Global vars
app = Flask(__name__)
CLIENT = None
DB = None
COLLECTION = None



@app.route('/')
def index():
    return API_DESCRIPTION

@app.route('/insertrecord/<string:student>', methods=['POST'])
def insertrecord(student):
    json_in = request.get_json(force = True)
    
    try:
        validate(json_in,SCHEMA)
    except:
        return "JSON NOT VALID !"
    json_in["student"] = student;
    COLLECTION.insert(json_in)
    return "Ok"


@app.route('/getrecords/<string:student>', methods=['GET'])
def consult(student):
    values = COLLECTION.find({"student" : student})
    if values.count() == 0:
        abort(404)
    l = list(values)
    for a in l:
        del a["_id"]
    return dumps(l)

if __name__ == '__main__':
    if len(sys.argv) != 2:
        print "Usage : "+sys.argv[0]+" mongo_instance_ip_address"
        exit(0)
    CLIENT = MongoClient(sys.argv[1], 27017)
    DB = CLIENT.labo
    COLLECTION = DB.records
    app.run(debug=False,host='0.0.0.0')
