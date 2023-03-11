<script>

export default {
    emits: ['rate', 'next', 'show'],
    props: {
        State: Object,
        state: Number,
        members: Object,
    },
    data() {
        return {
            scores: [0, 0.5, 1, 2, 3, 5, 8, 13, 20, 40, 100, "?", "☕"],
            controls: [{text:"Next", fn: this.next}, {text: "Show", fn: this.show}],
        }
    },
    methods: {
        rate(score) {
            this.$emit('rate', score)
        },
        next() {
            this.$emit('next')
        },
        show() {
            this.$emit('show')
        },
        memberScoreContent(id, score) {
            switch (this.state) {
                case this.State.VOTING:
                    if (id === 'local') {
                        return score || "🤔"
                    }
                    if (score) {
                        return "✅"
                    }
                    return "🤔"
                case this.State.FINISH:
                    if (score) {
                        return score
                    }
                    return "❌"
            }
        }
    },
    computed: {
        resultContent() {
            switch (this.state) {
                case this.State.VOTING:
                    return 'Click "show" to calculate an estimate'
                case this.State.FINISH:
                    let score = 0, c = 0
                    for (const id in this.members) {
                        const s = this.members[id].score
                        if (typeof s == 'number') {
                            score += s
                            c++
                        }
                    }
                    return score/c
            }
        }
    }
}
</script>

<template>
    <v-container>
        <!-- Score -->
        <v-row justify="center">
            <v-col class="d-flex justify-center" v-for="score in scores">
                <v-btn @click="rate(score)">
                    {{ score }}
                </v-btn>
            </v-col>
        </v-row>
        <!-- Controls -->
        <v-row>
            <v-col class="d-flex justify-center" v-for="control in controls">
                <v-btn @click="control.fn()">
                    {{ control.text }}
                </v-btn>
            </v-col>
        </v-row>
        <!-- Info -->
        <v-row>
            <v-col cols="12" sm="12" md="6">
                <v-row>
                    <v-col>
                        Members:
                    </v-col>
                </v-row>
                <v-row v-for="(member, id) in members" :key="id">
                    <v-card class="w-100">
                        <v-card-text>
                            <v-row align="center">
                                <v-col cols="10">
                                    {{ member.username || "Unknown" }}{{ id === 'local' ? "(Me)" : "" }}
                                </v-col>

                                <v-col cols="2">
                                    <v-card class="pa-1 text-center">
                                        {{ memberScoreContent(id, member.score) }}
                                    </v-card>
                                </v-col>
                            </v-row>
                        </v-card-text>
                    </v-card>
                </v-row>
            </v-col>
            <v-col>
                <v-row><v-col>Results:</v-col></v-row>
                <v-row>
                    <v-col class="text-center">
                        {{ resultContent }}
                    </v-col>
                </v-row>
            </v-col>
        </v-row>
    </v-container>
</template>