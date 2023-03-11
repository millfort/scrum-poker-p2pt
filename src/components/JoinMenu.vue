<script>

export default {
    emits: ['join'],
    data() {
        return {
            room: '',
            username: '',
        }
    },
    methods: {
        join() {
            if (this.room.trim() === '' || this.username.trim() === '') {
                return
            }
            this.$emit('join', this.room, this.username)
        }
    }
}
</script>
<template>
    <v-container>
        <v-text-field label="Room" v-model="room"></v-text-field>
        <v-text-field label="Username" v-model="username"></v-text-field>
        <v-btn @click="join">Join</v-btn>
    </v-container>
</template>