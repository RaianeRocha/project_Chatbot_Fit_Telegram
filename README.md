# project_Chatbot_Fit_Telegram
Construindo um ChatbotFit no Telegram com JavaScript e NodeJS

//pasta index.js

const TelegramBot = require('node-telegram-bot-api');
const  dialogflow = require('./dialogflow');
const youtube = require('./yotube');

const token = 'Colar token gerado';

const bot = new TelegramBot(token, { polling: true });

bot.on('message', async function (msg) {
    const chatId = msg.chat.id;
    console.log(msg.text);

    const dfResponse = await sendMessage(chatId.toString(), msg.text);

    let responseText = dfResponse.text;

    if (dfResponse.intent == 'Alimentacao Especifica') {
       responseText = await searchVideoURL(responseText, dfResponse.fields.alimentos.stringValue);
    }
    
    bot.sendMessage(chatId, await dfResponse.text);
});

// pasta Dialogflow.js

const dialogflow =require('dialogflow');
const configs = require('./dio-bot-fit.json');

const sessionClient = new dialogflow.SessionsClient({
    projectId: configs.project_id,
    credentials: {
        private_key: configs.private_key,
        client_email: configs.client_email
    }
});

async function sendMessage(chatId, message) {
    const sessionPath = sessionClient.sessionPath(configs.project_id, chatId);
    const request = {
        session: sessionPath,
        queryInput: {
            text: {
                text: message,
                languageCode: 'pt-BR'
            }
        }
    }

    const responses = await sessionClient.detectIntent(request);
    const result = responses[0].queryResult;
    return {
        text: result.fulfillmentText,
        intent: result.intent.displayName,
        fields: result.parameters.fields
    };
};

module.exports.sendMessage = sendMessage;


// pasta Youtube.js


const YouTube = require('youtube-node');
const config = require('./yt-config.json');

const youtube = new YouTube();
youtube.setKey(config.key);


    function searchVideoURL(message, queryText) {
        return new Promise((resolve, reject) => {
            youtube.search(`Alimentação Saudável ${queryText}`, 2, function(error, result) {
                if(!error) {
                    const videoIds = result.items.map((item) => item.id.videoId).filter(item => item);
                    const youtubeLinks = videoIds.map(videoId => `https://www.youtube.com/watch?v=${videoIds}`);
                    resolve(`${message} ${youtubeLinks.join(`, `)}`);
                } else {
                    reject('Deu erro!');
                }
            });
        });
    };
    
module.exports.searchVideoURL = searchVideoURL;







     
