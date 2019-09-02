<template>
  <div>
    <h1>GQL-Chat Vue</h1>
    <form @submit.prevent="sendMessage">
      <input v-model="messageText" />
      <button>Send</button>
    </form>
    <ul>
      <li v-for="message in messages" :key="message.id">
        {{ message.text }}
      </li>
    </ul>
  </div>
</template>

<script>
import gql from 'graphql-tag'

const LOAD_MESSAGES_QUERY = gql`
  query {
    messages {
      id
      text
    }
  }
`

const SEND_MESSAGE_MUTATION = gql`
  mutation($text: String!) {
    sendMessage(text: $text) {
      id
      text
    }
  }
`

const NEW_MESSAGE_SUBSCRIPTION = gql`
  subscription restaurantAdded {
    messageSent {
      id
      text
    }
  }
`

export default {
  name: 'Conversation',
  data() {
    return {
      messageText: '',
    }
  },
  methods: {
    sendMessage() {
      this.$apollo
        .mutate({
          mutation: SEND_MESSAGE_MUTATION,
          variables: {
            text: this.messageText,
          },
        })
        .then(() => {
          this.messageText = ''
        })
    },
  },
  apollo: {
    messages: {
      query: LOAD_MESSAGES_QUERY,
      subscribeToMore: {
        document: NEW_MESSAGE_SUBSCRIPTION,
        updateQuery: (previousResult, { subscriptionData }) => {
          const { messageSent } = subscriptionData.data
          return {
            messages: [...previousResult.messages, messageSent],
          }
        },
      },
    },
  },
}
</script>
