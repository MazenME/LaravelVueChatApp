<!-- crete simple chat UI markup in tailwind css -->
<template>
    <div class="flex flex-col h-full">
        <div class="flex-1 overflow-y-auto">
            <div v-for="message,index in messages" :key="index" class="flex flex-col space-y-4">
                <div v-if="message.sender_id!==currentUser.id" class="flex items-center justify-start space-y-2">
                    <div class="bg-gray-200 p-2 rounded-lg mt-3">
                        <p class="text-sm">{{ message.text }}</p>
                    </div>
                </div>
                <div v-else class="flex items-center justify-end">
                    <div class="bg-blue-500 text-white p-2 rounded-lg mt-3">
                        <p class="text-sm">{{ message.text }}</p>
                    </div>
                </div>
            </div>
        </div>
        <div class="flex items-center space-x-2 mt-14">
            <input v-model="newMessage" type="text" class="w-full p-2 border border-gray-300 rounded-lg" placeholder="Type your message here...">
            <button @click="sendMessage" class="bg-blue-500 text-white p-2 rounded-lg">Send</button>
        </div>
    </div>
</template>
<script setup>
    import { ref } from 'vue'
    import { onMounted } from 'vue';
    import axios from 'axios'
import Echo from 'laravel-echo';
    const props =defineProps({
        friend: {
            type: Object,
            required: true
        },
        currentUser: {
            type: Object,
            required: true
        }
    })
    const messages = ref([])

    const newMessage = ref('')
    const sendMessage = () => {
        if (newMessage.value.trim() !== '') {
            try{
                axios.post(`/message/${props.friend.id}`, {
                    message: newMessage.value,
                }).then(
                    (res) => {
                        messages.value.push(res.data)
                        newMessage.value = ''
                    }
                )
            }
            catch(e){
                console.log('error',e)
            }
        }
    }

    onMounted(()=> {
        axios.get(`/messages/${props.friend.id}`).then(
            (res) => {
                messages.value = res.data
                console.log('messages',messages.value)
            }
        )

        window.Echo.private(`chat.${props.currentUser.id}`)
        .listen('MessageSent', (e) => {
            console.log('message',e.message)
            messages.value.push(e.message)
        });
    })

</script>
