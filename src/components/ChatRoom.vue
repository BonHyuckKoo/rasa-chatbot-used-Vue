<template>
    <v-container>
        <div class="light-blue" id="chatWindow">
            <template v-for="item in items">
                <v-list-item v-if="item.type === 'response'" :key="item.id">
                    <i class="fas fa-robot" style="font-size: 36px; margin-right: 5px"></i>
                    <v-list-item-content>
                        <div class="questionChat__response" v-show="item.text">{{ item.text }}</div>
                        <div v-show="item.image">
                            <img :src="item.image" alt="" width="300px" height="300px">
                        </div>
                        <div v-show="item.buttons">
                            <v-simple-table dense id="buttonTable">
                                <template v-slot:default>
                                    <tbody>
                                        <tr
                                        v-for="button in item.buttons"
                                        :key="button.title"
                                        class="button__item"
                                        >
                                        <td v-on:click="clickButton(button.title)">{{ button.title }}</td>
                                        </tr>
                                    </tbody>
                                </template>
                            </v-simple-table>
                        </div>
                    </v-list-item-content>
                </v-list-item>
                <v-list-item v-else :key="item.id">
                    <v-list-item-content class="flex-row-reverse">
                        <div class="questionChat__question">{{ item.text }}</div>
                    </v-list-item-content>
                    <i class="fas fa-user-circle" style="font-size: 36px; margin-left: 5px;"></i>
                </v-list-item>
            </template>
        </div>
        <v-text-field v-model="textMessage" v-on:keyup.enter="sendMessage()"></v-text-field> 
    </v-container>
</template>

<script>
export default {
    data() {
        return {
            items: [],
            textMessage: "",
            msgIdx: 0
        }
    },
    updated() {
        const chatWindow = document.getElementById("chatWindow");
        chatWindow.scrollTop = chatWindow.scrollHeight
    },
    methods: {
        requestRasa: async function (data) {
            const requestOptions = {
                method: "POST", 
                headers: { "Content-Type": "application/json" },
                body: JSON.stringify(data)
            };
            const responseObj = await fetch("http://localhost:5005/webhooks/rest/webhook", requestOptions);
            const responseData = await responseObj.json()
            return responseData
        },
        responseFromRasa: function (responseData) {
            responseData.forEach(response => {
                this.items.push({
                    id: this.msgIdx++,
                    type: "response",
                    text: response.text,
                    image: response.image,
                    buttons: response.buttons
                })
            });
        },
        sendMessage: async function () {
            if (!this.textMessage) return;
            
            const data = {
                sender: "asasd", 
                message: this.textMessage 
            };
            this.items.push({
                id: this.msgIdx++,
                type: "request",
                text:  this.textMessage
            })

            this.textMessage = "";
            const responseData = await this.requestRasa(data)

            if (responseData.length) this.responseFromRasa(responseData);

        },
        clickButton: function (message) {
            const data = {
                sender: "asasd", 
                message: message 
            };
            this.items.push({
                id: this.msgIdx++,
                type: "request",
                text:  message
            })

            const responseData = this.requestRasa(data)
            if (responseData.length) this.responseFromRasa(responseData)
        }
    }
}
</script>

<style>
/* Hide scrollbar for Chrome, Safari and Opera */
#chatWindow::-webkit-scrollbar {
    display: none;
}

#chatWindow {
    height: 80vh;
    overflow-y: scroll;
}

.questionChat__response {
    text-align: center;
    background-color: white;
    padding: 10px 30px;
    max-width: 20vw;
    border-radius: 10px;
    word-break: keep-all;
}

.questionChat__question {
    text-align: center;
    background-color: white;
    padding: 10px 30px;
    max-width: 20vw;
    border-radius: 10px;
    word-break: break-all;
}

#buttonTable {
    width: 300px;
    text-align: center;
    border-radius: 10px;
}

.button__item {
    cursor: pointer;
}
</style>