<template>
<div id="weeklyProgramme">
  <v-container grid-list-md>
    <v-layout row wrap justify-space-between>
      <v-flex xs3>
        <v-layout>
          <Toolbar :data="doc" />
        </v-layout>
      </v-flex>

      <!-- <v-flex xs2></v-flex>
      <v-flex></v-flex>-->
      <v-flex xs9>
        <v-flex justify-end>
          <hr style="color:grey" />
        </v-flex>
        <v-layout wrap>
          <v-flex xs12>
            <h3 class="text-xs-center">HANDING AND TAKING-OVER CERTIFICATE</h3>

            <v-card>
              <br/>
              

                
              <v-card-text>
              <v-flex xs12>
                  <strong>OFFICIAL HANDING OVER :</strong> {{doc.body.officialHandingOver}}
                  </v-flex>
              
              <v-flex xs12>
                  <strong>LOCATION OF ASSETS :</strong> {{doc.body.locationOfAssets}}
                  </v-flex>

                  <v-flex xs12>
                  <strong>ASSET DESCRIPTION :</strong> {{doc.body.assetDescription}}
                  </v-flex>
              </v-card-text> 

              <v-card-title>
                <v-flex xs12 sm6>
                 <strong>ASSET NUMBER :</strong>{{doc.body.assetNumber}}
                  </v-flex>

                  <v-flex xs12 sm6>
                   <strong>SERIAL NUMBER :</strong>{{doc.body.serialNumber}}
                  </v-flex>

                  
               <v-flex xs12>
                  <strong>OFFICIAL TAKING OVER :</strong>{{doc.body.official}}
                  </v-flex>

                  <v-flex xs12>
                  <strong>NEW LOCATION OF ASSET :</strong>{{doc.body.locationOfAssets1}}
                  </v-flex>

                  <v-flex xs12>
                 <strong>REASON FOR HAND OVER :</strong>{{reasonForHandOver}}
                  </v-flex>


                  
                  
                  <p><strong> terms of the provisions of s42 of the PFMA(ACT 1,1999) we hereby certify that we have completed 
                  the verification of all Government Property on hand in accordance with balances of the ledger, or other records</strong></p>
                      <p><strong>We further certify that the list of discrepancies includes all items 
                          which have not hitherto been submitted for authority to adjust.</strong></p>

                  <v-flex xs12 sm6>
                      <strong>DATE OF HANDING / TAKING OVER :</strong>{{doc.body.date1}}
                </v-flex>


                <v-card-text>
           <v-flex
              v-for="signatory in doc.body.signatures"
              :key="signatory.SurName"
              xs12
              class="my-5"
            >
              <v-flex xs12>
                <h2>{{signatory.response}}</h2>
                <div v-html="signatory.comment"></div>
              </v-flex>
              <v-flex xs4>
                <img style="width:100%" :src="signatory.signature" alt />
              </v-flex>
              <v-layout xs12>
                <v-flex xs8 class="black--text ma-0 font-weight-bold text-sm-left">
                  <h3>{{signatory.SurName}}</h3>
                  <h4>
                    {{signatory.Designation}} |
                    <span class="grey--text">{{signatory.Department}}</span>
                  </h4>
                </v-flex>

                <v-flex xs4>
                  <p
                    class="text-xs-left"
                  >{{signatory.signatureDate | moment("(ddd) DD - MMM - YYYY")}}</p>
                  <hr />
                  <h4>Date</h4>
                </v-flex>
              </v-layout>
            </v-flex>

             </v-card-text>


              </v-card-title>

              


            </v-card>


            

          </v-flex>
        </v-layout>
      </v-flex>
    </v-layout>
  </v-container>
</div>
</template>

<script>
import Vue from "vue";
import Toolbar from "~/components/PreviewToolbar";
import SelectUsers from "~/components/SelectUsers";
import store from "~/store/store";
import Editor from "@tinymce/tinymce-vue";
import { signatureHelpers } from "~/services/helpers";
import VueSignaturePad from "vue-signature-pad";
import record from "~/services/docLog";

import { createDoc } from "~/services/DocsService";

Vue.use(VueSignaturePad);
Vue.use(Editor);
export default {
  name: "HandingTakingOverCertificate",
  components: {
    editor: Editor,
    Toolbar,
    SelectUsers
  },

  data() {
    return {
      menu1a: false,
      menu1b: false,
      menu2a: false,
      menu2b: false,
      menu3a: false,
      menu3b: false,
      menu4a: false,
      menu4b: false,

      headers: [
        {
          text: "ITEM No.",
          align: "left",
          sortable: false,
          value: "",
        //   width: "10%"
        },
        {
          text: "ITEM CHECKED",
          value: ""
        },
        {
          text: "YES / NO",
          value: "",
        //   width: "10%"
        },
        {
          text: "COMMENTS",
          value: ""
        },
        // {
        //   text: "TIME",
        //   value: "",
        //   width: "10%"
        // }
      ],

      user: {},
      users: [],
      loading: false,
      response: {},
      signatories: [],
      signature: "",
      ref: "",
      textInput: "",
      comment: "",
      action: {},

       actions: [
        { text: "For Approval", value: "approve", signLevel: 3 },
        { text: "For Recommendation", value: "recommend", signLevel: 2 },
        { text: "For Input", value: "input", signLevel: 1 },
        { text: "For Attention", value: "attention", signLevel: 0 }
      ],

      signer: {
        action: {}
      },
      min_height: 320
    };
  },
  computed: {
    doc() {
      return store.state.doc;
    },
    time() {
      return Date.now();
    },
    setAction(action) {
      return doc.action == action;
    }
  },
  watch: {
    doc(data) {
      console.log("We have data!", data);

      let pos = this.doc.body.signatures
        .map(function(e) {
          return e.EmailAdress;
        })
        .indexOf(store.state.user.EmailAdress);

      this.signer = this.doc.body.signatures[pos];
      if (this.signer) {
        this.action = this.respondText(this.signer.action);
      }
    }
  },
  methods: {
    // sign
    clear() {
      this.$refs.signaturePad.clearSignature();
      this.signature = null;
      this.doc.body.authorSignature = null;
      console.log(this.signature);
    },
    respondText(action) {
      if (action.value == "approve") {
        return {
          text: "Approve",
          value: "Approved",
          altText: "Not Approved",
          altValue: "Not Approved"
        };
      }
      if (action.value == "recommend") {
        return {
          text: "Recommended",
          value: "Recommended",
          altText: "Not Recommended",
          altValue: "Not Recommended"
        };
      }
    },
    saveSignature() {
      let pos = this.doc.body.signatures
        .map(function(e) {
          return e.EmailAdress;
        })
        .indexOf(store.state.user.EmailAdress);
      let user = this.doc.body.signatures[pos];
      user.signature = this.signature;
      user.signatureDate = Date.now();
      console.log(this.doc);
    },
    onEnd() {
      const { isEmpty, data } = this.$refs.signaturePad.saveSignature();
      console.log("=== End ===");
      this.signature = data;
      this.saveSignature()
    }
  },
  async created() {
    console.log(this.ref, this.$route.params.id);
    await store.dispatch("getDocById", this.$route.params.id);
    console.log(store.state.doc);
  }
};
</script>

<style>
.side-toolbar {
  position: fixed;
  max-width: 180px;
}

table {
  border-collapse: collapse;
}

td,
th {
  border: 1px solid #dddddd;
}

.signature-pad {
  max-width: 480px;
  background-color: #fff;
  border: 1px dotted black;
}

.terms_conditions {
  list-style: initial;
}
</style>
