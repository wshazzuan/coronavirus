# coronavirus
// This sample demonstrates handling intents from an Alexa skill using the Alexa Skills Kit SDK (v2).
// Please visit https://alexa.design/cookbook for additional examples on implementing slots, dialog management,
// session persistence, api calls, and more.
const Alexa = require('ask-sdk-core');
const https = require('https')
//----URL_CALLING_FUNCTION----
const getHttps = function(URL) {
    return new Promise((resolve, reject) => {
        const request = https.get(URL, response => {
            response.setEncoding('utf8');
            let returnData = '';
            if ((response.statusCode < 200 || response.statusCode >= 300) && (Number(response.statusCode) !== 404)){ //added && (Number(response.statusCode) !== 404
                return reject(new Error(`${response.statusCode}: ${response.req.getHeader('host')} ${response.req.path}`));
            }
            response.on('data', chunk => {
                returnData += chunk;
            });
            response.on('end', () => {
                resolve(returnData);
            });
            response.on('error', error => {
                reject(error);
            });
        });
        request.end();
    });
}

//----TIME_AND_DATE_FORMAT_FUNCTION----
function reformat_time(x){
    var t = Number(x.substring(11,13));
    var sign;
    if (t === 12){
        sign = "PM";
    }
    else if (t === 0){
        t = 12;
        sign = "PM";
    }
    else{
        sign = "AM"
    }
    return (String(t) + sign);
}

function reformat_date(x){
    const a = Number(x.substring(0,4)); //year
    const b = Number(x.substring(5,7)); //month2020-08-05
    const c = Number(x.substring(8,10)); //days
    const newdateformat = new Date(Date.UTC(a, b, c, 0, 0, 0));
    const days_full_name = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];
    const dayName = days_full_name[newdateformat.getDay()];
    const months_full_name = ['January', 'Febuary', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'November', 'December']
    const monthName = months_full_name[b-1];
    return (dayName + ", " + c + " " + monthName + " " + a);
}

/*function getMonthNumber() {
    var monthNumber
    getMonthNumber.mnths = getMonthNumber.mnths ||  ["january", "february", "march", "april", "may", "june", "july", "august", "september", "october", "november", "december"];
    return (getMonthNumber.mnths.indexOf(monthNumber.toLowerCase()) + 1);
}*/

//----TASK_NUMBER_ONE----
const GetTaskOneIntentHandler = {
    canHandle(handlerInput) {
        return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest'
            && Alexa.getIntentName(handlerInput.requestEnvelope) === 'GetTaskOneIntent';
    },
    async handle(handlerInput) {
         var number_result1 = 3 //default number
        var number = handlerInput.requestEnvelope.request.intent.slots.number.value //slot intent
        if (number !== undefined){              //if no number, follow default number
            number_result1 = Number(number)
        }
        if (number_result1>10){      //if user request more than 10, then only show 10
            number_result1=10
        }
        if(number_result1<3){        //if user request less than 3, then only show 3
            number_result1=3
        }
        var mortality_rate1 = [], country_name = [], i, j
        
        //simulate URL error by putting something like var URL = "https://api.covid19api.com/summary" 
        var URL = "https://api.covid19api.com/summary" 
        var response = await getHttps(URL)
        var data = JSON.parse(response)
        var speakOutput
        
        if (data.message === undefined) { //didn't have error message, meaning the above URL is valid API
            var rate1, w
            for (j = 0; j < number_result1; j++){
                mortality_rate1.push(-1)
                country_name.push('xyz')
                for (i = 0; i < data.Countries.length; i++) {
                    rate1 = data.Countries[i].TotalConfirmed
                    if (rate1 > mortality_rate1[j]) {
                        mortality_rate1[j] = rate1 
                        country_name[j] = data.Countries[i].Country
                        w = i
                    }
                }
                data.Countries[w].TotalConfirmed = -1
            }    
            if (number_result1 > 1){
                speakOutput = "As you can see, here are the top " + number_result1 + " countries with the highest total confirmed cases: " 
            }   
            else{
            speakOutput = "As you can see, this is the highest total confirmed cases"
            }
            for (j = 0; j < number_result1; j++){
                speakOutput =  speakOutput + (j+1) + " : " + country_name[j] + " with " + Number(mortality_rate1[j]) + " cases. "
            }
        }
        else {
            speakOutput = "There is something wrong here, have you check the URL?."
        }
        
        var URL_IFTTT = "https://maker.ifttt.com/trigger/corona_virus/with/key/o4fWg4abRyvQ9ynY5z6UQjIXJBgl0AgrbCu60luA470?value1=helix"
        URL_IFTTT = URL_IFTTT + speakOutput 
        https.get(URL_IFTTT)
        
        return handlerInput.responseBuilder
            .speak(speakOutput)
            .reprompt('Do you still want to continue?')
            .getResponse();
    }
};

//----TASK_NUMBER_TWO----
const GetTaskTwoIntentHandler = {
    canHandle(handlerInput) {
        return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest'
            && Alexa.getIntentName(handlerInput.requestEnvelope) === 'GetTaskTwoIntent';
    },
    async handle(handlerInput) {
         var number_result = 3 //default number
        var number = handlerInput.requestEnvelope.request.intent.slots.number.value //slot intent
        if (number !== undefined){              //if no number, follow default number
            number_result = Number(number)
        }
        if (number_result>10){      //if user request more than 10, then only show 10
            number_result=10
        }
        if(number_result<3){        //if user request less than 3, then only show 3
            number_result=3
        }
        var mortality_rate = [], country_name = [], i, j
        
        //simulate URL error by putting something like var URL = "https://api.covid19api.com/summaryx" 
        var URL = "https://api.covid19api.com/summary" 
        var response = await getHttps(URL)
        var data = JSON.parse(response)
        var speakOutput
        
        if (data.message === undefined) { //didn't have error message, meaning the above URL is valid API
            var rate, w
            for (j = 0; j < number_result; j++){
                mortality_rate.push(-1)
                country_name.push('xyz')
                for (i = 0; i < data.Countries.length; i++) {
                    rate = 100*data.Countries[i].TotalDeaths/data.Countries[i].TotalConfirmed
                    if ((rate > mortality_rate[j]) && (rate < 100)) {
                        mortality_rate[j] = rate 
                        country_name[j] = data.Countries[i].Country
                        w = i
                    }
                }
                data.Countries[w].TotalDeaths = -1
            }    
            if (number_result > 1){
                speakOutput = "As you can see, here are the top " + number_result + " countries with the highest mortality rate: "
            }   
            else{
            speakOutput = "As you can see, this is the country with highest mortality rate"
            }
            for (j = 0; j < number_result; j++){
                speakOutput = speakOutput + (j+1) + " : " + country_name[j] + " with " + Number(mortality_rate[j]).toFixed(2) + " percent. "
            }
        }
        else {
            speakOutput = "There is something wrong here, have you check the URL?."
        }
        
        var URL_IFTTT = "https://maker.ifttt.com/trigger/corona_virus/with/key/o4fWg4abRyvQ9ynY5z6UQjIXJBgl0AgrbCu60luA470?value1=helix"
        URL_IFTTT = URL_IFTTT + speakOutput 
        https.get(URL_IFTTT)
        
        return handlerInput.responseBuilder
            .speak(speakOutput)
            .reprompt('Do you still want to continue?')
            .getResponse();
    }
};

//----TASK_NUMBER_THREE----
const GetTaskThreeIntentHandler = {
    canHandle(handlerInput) {
        return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest'
            && Alexa.getIntentName(handlerInput.requestEnvelope) === 'GetTaskThreeIntent';
    },
    async handle(handlerInput) {
        var country = handlerInput.requestEnvelope.request.intent.slots.country.value
        var URL = "https://api.covid19api.com/total/country/" + country
        var response = await getHttps(URL)
        var data = JSON.parse(response)
        var d = data[data.length-1].Date
        var newtime = reformat_time(d)
        var newdate = reformat_date(d)
        var info1 = 100*data[data.length-1].Deaths/data[data.length-1].Confirmed
        var info2 = 100*data[data.length-1].Active/data[data.length-1].Confirmed
        
        if (data.message === undefined){
            var speakOutput = country + " on " + newdate + " at " + newtime + " has total case of " + data[data.length-1].Confirmed + ". "
            speakOutput = speakOutput + "With mortality rate of " + info1.toFixed(2) + " percent, total deaths of " + data[data.length-1].Deaths + " people. "
            speakOutput = speakOutput + "The total recover case are " + data[data.length-1].Recovered + " with recovery rate of " + info2.toFixed(2) + " percent."
        }
        else{
            speakOutput = "The data for the " + country + " is currently not available. Perhaps go and try for different country. "
        }
        
        var URL_IFTTT = "https://maker.ifttt.com/trigger/corona_virus/with/key/o4fWg4abRyvQ9ynY5z6UQjIXJBgl0AgrbCu60luA470?value1=helix"
        URL_IFTTT = URL_IFTTT + speakOutput 
        https.get(URL_IFTTT)
        
        return handlerInput.responseBuilder
            .speak(speakOutput)
            .reprompt('Do you still want to continue?')
            .getResponse();
    }
};

//----TASK_NUMBER_FOUR----
const GetTaskFourIntentHandler = {
    canHandle(handlerInput) {
        return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest'
            && Alexa.getIntentName(handlerInput.requestEnvelope) === 'GetTaskFourIntent';
    },
    async handle(handlerInput) {
        var country = handlerInput.requestEnvelope.request.intent.slots.country.value
        
        var reference_date = handlerInput.requestEnvelope.request.intent.slots.reference_date.value
        var reference_month = handlerInput.requestEnvelope.request.intent.slots.reference_month.value
        var reference_year = handlerInput.requestEnvelope.request.intent.slots.reference_year.value
        
        var latest_date = handlerInput.requestEnvelope.request.intent.slots.latest_date.value
        var latest_month = handlerInput.requestEnvelope.request.intent.slots.latest_month.value
        var latest_year = handlerInput.requestEnvelope.request.intent.slots.latest_year.value
        
        latest_year = "2020"
        reference_year = "2020"
        
        var monthNum = ["january", "febuary", "march", "april", "may", "june", "july", "august", "september", "november", "december"]
        //change month into integer
        var i
        for( i = 0; i < 12; i++){
            if(reference_month.toLowerCase() === monthNum[i]){
                reference_month = "0" + (i+1)
                reference_month = reference_month.substr(-2)
                break
            }
        }
        for( i = 0; i < 12; i++){
            if(latest_month.toLowerCase() === monthNum[i]){
                latest_month = "0" + (i+1)
                latest_month = latest_month.substr(-2)
                break
            }
        }
        
        //format corona virus '2020-01-22T00:00:00Z' hence we need to arrange the string of the date
        var formatreference_date = ("0"+ reference_date).slice(-2)
        var formatlatest_date = ("0"+ latest_date).slice(-2)
        var reference = reference_year + "-" + reference_month + "-" + formatreference_date.substr(-2) + "T00:00:00Z"
        var latest = latest_year + "-" + latest_month + "-" + formatlatest_date.substr(-2) + "T00:00:00Z"
        
        var URL = "https://api.covid19api.com/total/country/" + country + "?from=" + reference + "&to=" + latest
        var response = await getHttps(URL)
        var data = JSON.parse(response)
        
        var d = data[data.length-1].Date
        var e = data[0].Date
        var newtimed = reformat_time(d)
        var newdated = reformat_date(d)
        var newtimee = reformat_time(e)
        var newdatee = reformat_date(e)
        
        var Info1 = data[data.length-1].Confirmed-data[0].Confirmed//total confirmed case
        var Info2 = data[data.length-1].Deaths-data[0].Deaths//total deaths
        var Info3 = data[data.length-1].Recovered-data[0].Recovered //total recovered
        var Info4 = data[data.length-1].Active-data[0].Active //total active 
        var Info5 = 100*(Info2/Info1)//mortality rate
        if (Info5===Infinity){
            Info5 = "0"
        }
        var Info6 = 100*(Info4/Info1)//recovery rate
        
        //var speakOutput = latest + "," + reference
        var speakOutput = "On " + country + " between " + newdatee + " and " + newdated + ": Increase of " + Info1 + " Confirmed Case, "
        speakOutput = speakOutput + "and total mortality of " + Info2 + " deaths, with " + Info3 + " recovered cases, with " + Info4 + " new case. "
        speakOutput = speakOutput + "The mortality rate are " + Info5.toFixed(2) + " percent and recovery rate are " + Info6.toFixed(2) + " percent."
        
        var URL_IFTTT = "https://maker.ifttt.com/trigger/corona_virus/with/key/o4fWg4abRyvQ9ynY5z6UQjIXJBgl0AgrbCu60luA470?value1=helix"
        URL_IFTTT = URL_IFTTT + speakOutput 
        https.get(URL_IFTTT)
        
        return handlerInput.responseBuilder
            .speak(speakOutput)
            //.reprompt('add a reprompt if you want to keep the session open for the user to respond')
            .getResponse();
    }
};
//----BASIC_ONE----
const LaunchRequestHandler = {
    canHandle(handlerInput) {
        return Alexa.getRequestType(handlerInput.requestEnvelope) === 'LaunchRequest';
    },
    handle(handlerInput) {
        const speakOutput = 'Welcome to Corona virus project my lovely friends';
        var URL_IFTTT = "https://maker.ifttt.com/trigger/corona_virus/with/key/o4fWg4abRyvQ9ynY5z6UQjIXJBgl0AgrbCu60luA470?value1=helix"
        URL_IFTTT = URL_IFTTT + speakOutput 
        https.get(URL_IFTTT)
        return handlerInput.responseBuilder
            .speak(speakOutput)
            .reprompt(speakOutput)
            .getResponse();
    }
};
const HelloWorldIntentHandler = {
    canHandle(handlerInput) {
        return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest'
            && Alexa.getIntentName(handlerInput.requestEnvelope) === 'HelloWorldIntent';
    },
    handle(handlerInput) {
        const speakOutput = 'Hello Everyone!';
        return handlerInput.responseBuilder
            .speak(speakOutput)
            //.reprompt('add a reprompt if you want to keep the session open for the user to respond')
            .getResponse();
    }
};
const HelpIntentHandler = {
    canHandle(handlerInput) {
        return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest'
            && Alexa.getIntentName(handlerInput.requestEnvelope) === 'AMAZON.HelpIntent';
    },
    handle(handlerInput) {
        const speakOutput = 'You can say hello to me! How can I help?';

        return handlerInput.responseBuilder
            .speak(speakOutput)
            .reprompt(speakOutput)
            .getResponse();
    }
};
const CancelAndStopIntentHandler = {
    canHandle(handlerInput) {
        return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest'
            && (Alexa.getIntentName(handlerInput.requestEnvelope) === 'AMAZON.CancelIntent'
                || Alexa.getIntentName(handlerInput.requestEnvelope) === 'AMAZON.StopIntent');
    },
    handle(handlerInput) {
        const speakOutput = 'Goodbye!';
        return handlerInput.responseBuilder
            .speak(speakOutput)
            .getResponse();
    }
};
const SessionEndedRequestHandler = {
    canHandle(handlerInput) {
        return Alexa.getRequestType(handlerInput.requestEnvelope) === 'SessionEndedRequest';
    },
    handle(handlerInput) {
        // Any cleanup logic goes here.
        return handlerInput.responseBuilder.getResponse();
    }
};

// The intent reflector is used for interaction model testing and debugging.
// It will simply repeat the intent the user said. You can create custom handlers
// for your intents by defining them above, then also adding them to the request
// handler chain below.
const IntentReflectorHandler = {
    canHandle(handlerInput) {
        return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest';
    },
    handle(handlerInput) {
        const intentName = Alexa.getIntentName(handlerInput.requestEnvelope);
        const speakOutput = `You just triggered ${intentName}`;

        return handlerInput.responseBuilder
            .speak(speakOutput)
            //.reprompt('add a reprompt if you want to keep the session open for the user to respond')
            .getResponse();
    }
};

// Generic error handling to capture any syntax or routing errors. If you receive an error
// stating the request handler chain is not found, you have not implemented a handler for
// the intent being invoked or included it in the skill builder below.
const ErrorHandler = {
    canHandle() {
        return true;
    },
    handle(handlerInput, error) {
        console.log(`~~~~ Error handled: ${error.stack}`);
        const speakOutput = `Sorry, I had trouble doing what you asked. Please try again.`;

        return handlerInput.responseBuilder
            .speak(speakOutput)
            .reprompt(speakOutput)
            .getResponse();
    }
};

// The SkillBuilder acts as the entry point for your skill, routing all request and response
// payloads to the handlers above. Make sure any new handlers or interceptors you've
// defined are included below. The order matters - they're processed top to bottom.
exports.handler = Alexa.SkillBuilders.custom()
    .addRequestHandlers(
        LaunchRequestHandler,
        HelloWorldIntentHandler,
        GetTaskOneIntentHandler,
        GetTaskTwoIntentHandler,
        GetTaskThreeIntentHandler,
        GetTaskFourIntentHandler,
        HelpIntentHandler,
        CancelAndStopIntentHandler,
        SessionEndedRequestHandler,
        IntentReflectorHandler, // make sure IntentReflectorHandler is last so it doesn't override your custom intent handlers
    )
    .addErrorHandlers(
        ErrorHandler,
    )
    .lambda();
