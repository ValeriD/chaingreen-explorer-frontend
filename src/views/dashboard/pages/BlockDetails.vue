<template>
  <v-container
    id='block-details'
    fluid
    tag='section'
  >
   <!-- Loading Screen -->
  <LoadingScreen v-if="isLoading"/>

  <!-- Screen when data loaded -->
  <template v-if="!isLoading">
    <v-row>
        <v-chip
          color="orange"
          rounded
          large
          text-color="white"
          align="left"
          >
          <div class="font-weight-regular display-1">Block Hash: {{ block_hash | formatStringLength($vuetify.breakpoint.mobile) }}</div>
          </v-chip>
    </v-row>
    <v-row>
        <v-col
          cols='12'
          sm='6'
          lg='3'
        >
          <base-material-stats-card
            color='green'
            icon='mdi-arrange-send-to-back'
            title='Block height'
            :value='block_height | formatNumber'
          />
        </v-col>

        <v-col
          cols='12'
          sm='6'
          lg='3'
        >
          <base-material-stats-card
          class="font-weight-light"
            color='orange'
            icon='mdi-clock-time-four-outline'
            title='Date and time'
            :value='date_time'
          />
        </v-col>

        <v-col
          cols='12'
          sm='6'
          lg='3'
        >
          <base-material-stats-card
            color='green'
            icon='mdi-circle-multiple-outline'
            title='TX Amount'
            :value='transactions_amount | formatCurrency'
          />
        </v-col>

        <v-col
          cols='12'
          sm='6'
          lg='3'
        >
          <base-material-stats-card
            color='purple'
            icon='mdi-cached'
            title='Number of transactions'
            :value='number_of_transactions | formatNumber'
          />
        </v-col>

        <v-col
          cols='12'
          md='12'
        >
        </v-col>
      </v-row>
      <v-row
        align='center'
        justify='center'
      >
        <v-col
          cols='12'
          md='12'
        >
          <base-material-card
            icon='mdi-clipboard-text'
            title='Transactions'
            class='px-5 py-3'
          >
            <v-card-text>
              <v-data-table
                :headers='headers'
                :items='items'
                @click:row="handleRowClick"
              >
              <template v-slot:[`item.transaction_id`]="{ value }">
                <div>
                  <a>{{ value | formatStringLength($vuetify.breakpoint.mobile) }}</a>
                </div>
              </template>
              <template v-slot:[`item.receiver`]="{ value }">
                <div>
                  <a @click.stop="handleRowClick" @click="handleColumnClick(value)">{{ value | formatStringLength($vuetify.breakpoint.mobile) }}</a>
                </div>
              </template>
              <template v-slot:[`item.sender`]="{ value }">
                <div>
                  <a @click.stop="handleRowClick" @click="handleColumnClick(value)">{{ value | formatStringLength($vuetify.breakpoint.mobile) }}</a>
                </div>
              </template>
              <template v-slot:[`item.amount`]="{ value }">
                <div>
                  {{ value | formatCurrency }}
                </div>
              </template>
              </v-data-table>
            </v-card-text>
          </base-material-card>
        </v-col>
      </v-row>
  </template>
   </v-container>
</template>

<script>
    import LoadingScreen from '../components/Loading'

    import { mapState, mapMutations } from 'vuex'

    export default {
        name: 'BlockDetails',
        components:{
          LoadingScreen
        },
        data : () => ({
            headers: [
              {
                sortable: false,
                text: 'Transaction ID',
                value: 'transaction_id',
              },
              {
                sortable: false,
                text: 'Sender',
                value: 'sender',
                align: 'left',
              },
              {
                sortable: false,
                text: 'Receiver',
                value: 'receiver',
                align: 'left',
              },
              {
                sortable: false,
                text: 'Amount',
                value: 'amount',
                align: 'left',
              },
            ],
            items: [],
            block_hash: '',
            block_height: '',
            date_time: '',
            transactions_amount: '',
            number_of_transactions: '',
            isLoading: true,
        }),
        computed: {
        ...mapState(['searchResult']),
        query(){
          return this.$route.query.block_hash;
        }
        },
        methods: {
            ...mapMutations({
                setSearch: 'SET_SEARCH_RESULT'
            }),
            setValues: function(block){
              this.block_hash = this.$route.query.block_hash
              this.block_height = block.reward_chain_block.height.toString()
              this.date_time = block.foliage_transaction_block? new Date(block.foliage_transaction_block.timestamp*1000).toString().slice(3, 24) : "No time info";
              this.transactions_amount = block.foliage_transaction_block? block.transactions_info.amount : "0";
              if(block.foliage_transaction_block){
                //TODO add more fields to handle click
                block.transactions_info.transactions.forEach(t => {
                  this.items.push({
                    transaction_id: t.transaction_id,
                    date: new Date( t.created_at).toString().slice(3, 24),
                    sender: t.sender,
                    receiver: t.receiver,
                    amount: t.amount.toString(),
                  })
                })
                this.number_of_transactions = this.items.length.toString()
              }else{
                this.number_of_transactions = '0';
              }
            },
            handleQueryChange: async function(){
              let block;
              if(this.searchResult && this.searchResult.block){
                block = this.searchResult.block;
              }else{
                block = (await this.axios.get('get_block', {params: {hash: this.$route.query.block_hash}})).data.block
              }
              this.setValues(block);
            },
             handleRowClick(item, e){
              if(e){
              this.$router.push({ path:'/transactions/details', query:{transaction_id: item.transaction_id}})
              }
            },
            handleColumnClick(address){
              this.redirect('/addresses/details/', {address: address});
            }
        },
        async mounted () {
            let block;
            if(this.searchResult && this.searchResult.block){
              block = this.searchResult.block;
            }else{
                block = (await this.axios.get('get_block', {params: {hash: this.$route.query.block_hash}})).data.block
            }
            this.setValues(block)
            this.isLoading = false;
        },
        watch:{
          query(){
            this.handleQueryChange();
          }
        }
    }
</script>
