<template>
  <div>

    <b-panel collapsible :open="!isClosed" has-custom-template>
      <span slot="header">
        <span>
          <b-taglist attached style="float: right">
            <b-tag v-if="isValueTransfer" type="is-dark" style="float: right">Value</b-tag>
            <b-tag :type="bundleStatus | toStatusType" style="float: right">{{ bundleStatus | toStatus}}</b-tag>
          </b-taglist>
        </span>
        <span class="title is-5 is-marginless" style="display:inline-block;">Bundle</span>
        <br>
        <span class="subtitle is-6" style="display:inline-block; width: calc(100% - 24px)">{{ bundleHash }}</span>
      </span>

      <div class="box">
        <div class="panel-block txrow">
          <h1 class="title is-5">From:</h1>
        </div>
        <search-tx :results="inputTxs" :iota="iota" :overallStatus="bundleStatus" :inBundle="true"></search-tx>
        <div class="panel-block txrow"><br></div>
        <div class="panel-block txrow">
          <h1 class="title is-5">To:</h1>
        </div>
        <search-tx :results="outputTxs" :iota="iota" :overallStatus="bundleStatus" :inBundle="true"></search-tx>
      </div>

      <div class="box" v-show="combineMessage.length > 0">
        <div class="panel-block txrow">
          <h1 class="title is-5">Message:</h1>
        </div>
        <pre style="white-space: pre-wrap;">{{ combineMessage }}</pre>
      </div>
    </b-panel>

  </div>
</template>

<script>
  import SearchTx from './tx'
  import TryteCodec from 'tryte-utf8-json-codec'

  export default {
    name: 'search-bundle',
    props: ['iota', 'results', 'isClosed', 'address'],
    components: {
      SearchTx
    },
    computed: {
      bundleHash () {
        return this.results && this.results[0] ? this.results[0].bundle : ''
      },
      bundleStatus () {
        return this.asyncTxStatus && this.asyncTxStatus[0]
      },
      outputTxs () {
        return this.results.filter(tx => tx.value > 0)
      },
      inputTxs () {
        return this.results.filter(tx => tx.value <= 0)
      },
      isValueTransfer () {
        return this.results.some(tx => tx.value !== 0)
      },
      combineMessage() {
        try {

          if (this.results.length == 0) {
            return '';
          }

          var data = "";
          var results = this.results.slice().reverse();
          for (var i = 0; i < results.length; i++) {
            data += results[i].signatureMessageFragment;
          }

          const text = TryteCodec.utf8StringFromTrytes(data);
          const json = JSON.parse(text);

          return JSON.stringify(json, null, 2);
        } catch (e) {
          console.log('error', e.message);
          return ''
        }
      }
    },
    filters: {
      toStatus (tx) {
        if (typeof tx === 'undefined') {
          return 'Unknown'
        } else {
          return tx ? 'Confirmed' : 'Pending'
        }
      },
      toStatusType (tx) {
        if (typeof tx === 'undefined') {
          return 'is-warning'
        } else {
          return tx ? 'is-info' : 'is-warning'
        }
      }
    },
    asyncComputed: {
      asyncTxStatus: {
        lazy: true,
        get () {
          return new Promise((resolve, reject) => {
            if (!this.results || this.results.length === 0) {
              return resolve([])
            } else {
              this.iota.api.getLatestInclusion([this.results[0].hash], (err, res) => {
                if (err) {
                  return resolve()
                }
                return resolve(res)
              })
            }
          })
        },
        default: []
      }
    }
  }
</script>

<style>
  .txrow {
    display: block;
  }

  .tag {
    font-size: 0.9rem;
  }
</style>
