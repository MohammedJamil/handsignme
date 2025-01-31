<template>
  <div v-if="docInfo !== null" class="sign row">
    <div class="row">
      <div v-if="err" class="alert alert-danger">
        {{err}}
      </div>
    </div>
    <div class="col mx-2">
      <PDFViewer v-if="file !== null" ref="viewer"
        :config="docInfo.configuration.data"
        :showOtherSignatures="docInfo.configuration.showOtherSignatures"
        :otherSignatories="docInfo.otherSignatories"
        :signatory="docInfo.signatory"
        :signMode="true"
        :src="file"/>
    </div>
    <form @submit="signDocument" class="col d-flex align-items-center my-4">
      <div v-if="docInfo != null" class="col mx-2 justify-content-center">
        <div v-for="(step, index) in steps" :key="index" class="row d-flex">
          <p>
            {{ index + 1 }}.
            <span v-if="step.signed">A signé p.{{ step.page }}</span>
            <span v-else>Doit signer p.{{ step.page }}</span>
            <i v-if="step.signed" class="m-2 bi-check-lg" style="color: green;" aria-hidden="true"/>
            <i v-else class="m-2 bi-x" style="color: red;" aria-hidden="true"/>
          </p>
        </div>
        <div class="row">
          <button type="submit" class="btn btn-success">Definitive sign</button>
        </div>
      </div>
      <div v-if="docInfo != null" class="col mx-2">
        <div class="row d-flex">
          <p>
            <i class="m-2 bi-check-circle" style="color: green;" aria-hidden="true"/>
            Integrity of the document verified under identifier {{ docInfo.hash }}
          </p>
        </div>
      </div>
    </form>
  </div>
</template>

<script>
// @ is an alias to /src
import PDFViewer from '@/components/PDFViewer.vue'
import http from '@/http-common'

export default {
  name: 'SignView',
  components: {
    PDFViewer
  },
  data () {
    return {
      docInfo: null,
      file: null,
      steps: [
        { page: 1, signed: true },
        { page: 2, signed: false }
      ],
      err: undefined
    }
  },
  methods: {
    signDocument (e) {
      const vm = this

      e.preventDefault()
      e.stopPropagation()

      http.post('/api/pdf/sign', {
        data: vm.$refs.viewer.signatureData()
      },
      {
        headers: {
          Authorization: vm.$route.params.token
        }
      })
        .then(res => {
          vm.$router.push('/sign/success')
        })
        .catch(err => {
          vm.err = err.response.data.error.msg
        })
    }
  },
  mounted () {
    const vm = this

    http.get('/api/pdf/info', {
      headers: {
        Authorization: vm.$route.params.token
      }
    })
      .then(res => {
        vm.docInfo = res.data
        if (vm.docInfo.signatory.signed) {
          vm.$router.push('/sign/error')
        }
      })
      .catch(err => {
        console.log(err.response)
      })

    http.get('/api/pdf/file', {
      responseType: 'blob',
      headers: {
        Authorization: vm.$route.params.token
      }
    })
      .then(res => {
        const blob = new Blob([res.data], { type: 'application/pdf' })
        vm.file = window.URL.createObjectURL(blob)
      })
      .catch(err => {
        console.log(err.response)
      })
  }
}
</script>
