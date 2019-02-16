# Maliba
DSRC Networks Congestion
/* To simulate I just launched function to create two network
  Initiate and declare variable
  Car or Vehicles
  */
var incomingCarSimulator = null;  // initiate to zero
var leavingCarSimulator = null;
var segment = null;
var carMessageLength = 400; 
var sendDataSize = 32; // 32 kbps 
var incomingCarInterval = 100;
var leavingCarInterval = 2000;
var incominCarIntervalWithCongestion = 190;
var sentDataInterval = 1000;
var receiveDataInterval = 1000;
var beginingTime = 0;
var endingTime = 0;
var freeflow;

var launchSimulator = function() {
 console.log("Launching Simulator");

 // Two network DSRC and LTE
 var networkDSRC = new Network({
   "dataRate": 6,
   "type": "DSRC",
   "isCongested": false // No congestion at the begining
 });
 var networkLTE = new Network({
   "dataRate": 6,
   "type": "LTE",
   "isCongested": false // No congestion at the begining
 });
 // Create a segment AB in whcih vehicles or meaages flow 
 segment = new Segment({
   "length": 1, // 1 mile
   "cars": [],
   "density": [],
   "meanSpeed":[],
   "flow": [],
   "network": [networkDSRC, networkLTE]
 });

  // Order of vehicles
 // Traffic congestion begins when density equals to 125, there will be some leaving vehicles 
 console.log(segment); 
 var carAutoIndex = 1;
 beginingTime = new Date();

 incomingCarSimulator = window.setInterval(function() {
  if (segment.getCurrentDensity() == 187) {
    clearInterval(incomingCarSimulator);
    incomingCarSimulator = window.setInterval(function() {
      var newCar = new Car({
         "order": carAutoIndex++,
         "segment": segment,
         "sendDataSize": 32  /////////////////////
       });
       segment.comingCars(newCar);
       segment.displayNewMessage();

     }, 210);
  } else {
     var newCar = new Car({
       "order": carAutoIndex++,
       "segment": segment,
       "sendDataSize": 32  /////////////////////
     });
     segment.comingCars(newCar);
     segment.displayNewMessage();
   }

 }, 50);

 leavingCarSimulator = window.setInterval(function() {
   segment.leavingCar();
 }, 160);
};
// Stop Simulator to the number of vehicles the users desire
var stopSimulator = function() {
 console.log("Stoping the simulator");
console.log(segment);
 segment.disable();
 clearInterval(incomingCarSimulator);
 clearInterval(leavingCarSimulator);

  endingTime = new Date();
  console.log("Stimulation duration in milliseconds is = "+ (endingTime - beginingTime));
};

/*Car Class
 Functions and parameters of each vehicle, "Car Class"
Functions and parameters of each vehicle
Set interval variable so we can stop sending data after the stumulation is stopped or the car leaves the segment.

One parameter of the vehicle is the number of BSM Message Sent. 
DSRC Channel Status BY (Strom, E. G. (2013, November). On 20 MHz channel spacing for V2X communication based on 802.11 OFDM.)
Maximum Capacity of DSRC Channel allowing vehicles to communicate without No DATA LOST. When the channnel is
getting close to its maximum capacity there will a tiny lost of data and become significant after 187 vehicles in segment AB. 

*/
var Car = function(params) {
  this.dataSent = [];
  this.receivedData = [];
  this.channelInUse = "";
  this.sentDataTimer = null; //Set interval variable so we can stop sending data after the stumulation is stopped or the car leaves the segment.
  this.receivedDataTimer = null;
 for (var param in params) {
   this[param] = params[param];
 }
};

/*
y
----------------------------
----------------------------
----------------------------x=vt
*/
/*
Find the position of all vehicles in order to know estblish a graph and ppropcess a routing method.
This is where the energy consumption concept begins. 
*/

Car.prototype.getGPSPosition = function() {
  var currentTime = new Date();
  var duration = currentTime - this.startingMoveDate;
  return [this.segment.length / leavingCarInterval * duration, 0]
};

Car.prototype.sendData = function() {
  this.startingMoveDate = new Date();
 this.sentDataTimer = window.setInterval(function() {
 this.takePath();
    if (this.segment.getCurrentDensity() <= 185){
     this.dataSent.push(10);

    }else if (this.segment.getCurrentDensity() <= 187){
    
      this.dataSent.push(10 - 2*Math.random());
    }

    else {
          this.dataSent.push(Math.random()*10);
    }
 }.bind(this), sentDataInterval);
};
