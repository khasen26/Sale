<template>
  <v-card>
    <v-card-title>{{ $t("sale.card") }}</v-card-title>
    <v-card-text>
      <v-timeline align-top dense>
        <!-- Шаг 1 - Параметры отпуска -->
        <v-timeline-item
          v-if="steps['Шаг 1 - Выбор параметров отпуска'].show"
          small
        >
          <span>{{ $t("sale.params") }}:</span>
          <v-flex xs12 sm12 md12>
            <v-select
              dense
              v-model="saleCategory.select"
              :items="saleCategory.items"
              item-text="name"
              item-value="id"
              :label="$t('sale.saleсategory.label')"
              persistent-hint
              return-object
              single-line
            >
            </v-select>
          </v-flex>
          <v-flex xs12 sm12 md12>
            <v-select
              disabled
              dens
              v-model="productCategory.select"
              :items="productCategory.items"
              item-text="name"
              item-value="id"
              :label="$t('sale.productсategory.label')"
              persistent-hint
              return-object
              single-line
            >
            </v-select>
          </v-flex>
        </v-timeline-item>
        <!-- Шаг 2 - Проверка документов покупателя -->
        <v-timeline-item
          small
          v-if="
            saleCategory.select.code !== 'Free' &&
            steps['Шаг 2 - Запрос баланса на основании документов'].show
          "
        >
          <div>
            <span>{{ $t("sale.docs") }}:</span>
            <div v-if="saleCategory.select.code == 'ByDriverLicense'">
              <v-form v-model="isFormValid">
                <v-flex
                  xs12
                  sm12
                  md12
                  class="mt-3 pr-1"
                  v-for="documentType in ['ИИН', 'ВУ']"
                  :key="`document-type${documentType}`"
                >
                  <v-text-field
                    dense
                    v-model="document[documentType].value"
                    :label="document[documentType].label()"
                    :rules="document[documentType].rules"
                    hide-details="auto"
                  >
                  </v-text-field>
                </v-flex>
              </v-form>
            </div>
             
          
              <v-row >
                <v-col><v-select v-model="selectedOption" label="Выберите способ получения данных" :items="items" outlined>
                  <option :value="option">{{ option }}</option></v-select></v-col>
              </v-row>
              


              
            <div v-if="saleCategory.select.code == 'ForCargoCar'">
              <div>
                <v-switch
                  v-model="switchCargoCarDocumentPack"
                  :label="cargoCarDocumentPackLabel"
                ></v-switch>
              </div>

              <div v-if="switchCargoCarDocumentPack">
                <v-form v-model="isFormValid">
                  <v-flex
                    xs12
                    sm12
                    md12
                    class="mt-3 pr-1"
                    v-for="documentType in ['ГРНЗ', 'СРТС']"
                    :key="`document-type${documentType}`"
                   
                  >
                  
                    <v-text-field
                      dense
                      v-model="document[documentType].value"
                      :label="document[documentType].label()"
                      :rules="document[documentType].rules"
                      hide-details="auto"
                    >
                    </v-text-field>
                  </v-flex>
                </v-form>
              </div>
              <div v-else>
                <v-form v-model="isFormValid">
                  <v-flex
                    xs12
                    sm12
                    md12
                    class="mt-3 pr-1"
                    v-for="documentType in ['ИИН','ГРНЗ', 'БИН']"
                    :key="`document-type${documentType}`"
                  >
                    
                    <v-text-field
                      dense
                      v-model="document[documentType].value"
                      :label="document[documentType].label()"
                      :rules="document[documentType].rules"
                      hide-details="auto"
                    >
                    </v-text-field>
                  </v-flex>
                </v-form>
              </div>
            </div>
            <div v-if="saleCategory.select.code == 'ForFuelCard'">
              <!-- продажи по топливной карте жанарммай-->
              <v-form v-model="isFormValid">
                <v-flex
                  xs12
                  sm12
                  md12
                  class="mt-3 pr-1"
                  v-for="documentType in ['ТК']"
                  :key="`document-type${documentType}`"
                >
                  <v-text-field
                    dense
                    v-model="document[documentType].value"
                    :label="document[documentType].label()"
                    :rules="document[documentType].rules"
                    hide-details="auto"
                  >
                  </v-text-field>
                </v-flex>
              </v-form>
            </div>
            <div v-if="saleCategory.select.code == 'ForOnTimeCode'">
              <!-- продажи по одноразовому коду -->
              <v-form v-model="isFormValid">
                <v-flex
                  xs12
                  sm12
                  md12
                  class="mt-3 pr-1"
                  v-for="documentType in ['ОК']"
                  :key="`document-type${documentType}`"
                >
                  <v-text-field
                    dense
                    v-model="document[documentType].value"
                    :label="document[documentType].label()"
                    :rules="document[documentType].rules"
                    hide-details="auto"
                  >
                  </v-text-field>
                </v-flex>
              </v-form>
            </div>

            <!-- Запросить баланс потребителя на основании предоставленных документов -->
            <v-row justify="space-between" class="pr-1">
              <v-btn
                text
                class="mt-3"
                @click="requestConsumerBalance"
                :disabled="requestConsumerBalanceButtonDisabled || !isFormValid"
              >
                {{ $t("sale.requestbalance") }}
              </v-btn>
              <v-btn text class="mt-3" @click="closeSaleCard">
                {{ $t("sale.cancel") }}
              </v-btn>
            </v-row>
          </div>
        </v-timeline-item>
        <!-- Шаг 3 - Реализация нефтепродукта -->
        <v-timeline-item
          small
          v-if="
            steps['Шаг 3 - Отпуск нефтепродуктов'].show ||
            saleCategory.select.code == 'Free'
          "
        >
          <div>
            <span>{{ $t("sale.confirm") }}:</span>
            <v-flex xs12 sm12 md12 class="mt-3 pr-1">
              <div v-if="saleCategory.select.code == 'Free'">
                <v-text-field
                  dense
                  type="number"
                  step="0.1"
                  :min="0"
                  v-model="released.current"
                  :rules="[
                    (v) => !!v || 'Обязательное поле',
                    (v) => v >= 1 || 'Не меньше одного литра',
                    (v) => v <= 2000 || 'Не более 2000 литров',
                    (v) =>
                      /^\d*.\d{1}$/.test(v) ||
                      /^\d*$/.test(v) ||
                      'Не более одного знака после запятой',
                  ]"
                  :label="$t('sale.release.label')"
                  :prefix="$t('sale.release.prefix')"
                  :suffix="$t('sale.release.suffix')"
                  hide-details="auto"
                  class="mt-5"
                  @keyup="setDecimalPlace"
                >
                </v-text-field>
              </div>
              <div v-else>
                <v-row dense>
                  <v-col cols="12">
                    <div>
                      <p>
                        {{ $t("sale.limits.daily") }}:
                        {{ limits.daily }}
                      </p>
                      <p>
                        {{ $t("sale.limits.balance") }}:
                        {{ limits.balance }}
                      </p>
                    </div>
                  </v-col>
                </v-row>
                <!-- <v-flex
                  xs12
                  sm12
                  md12
                  class="mt-5 pr-1"
                  v-for="documentType in ['ТК']"
                  :key="`document-type${documentType}`"
                > -->
                <v-text-field
                  v-if="saleCategory.select.code == 'ForFuelCard'"
                  dense
                  v-model="smsCode"
                  label="СМС Код"
                  :rules="[(v) => /^\d{4}$/.test(v) || 'Не более 4 цифр']"
                  hide-details="auto"
                >
                </v-text-field>

                <!-- </v-flex> -->
                <v-text-field
                  dense
                  type="number"
                  step="0.1"
                  :min="0"
                  :counter="
                    saleCategory.select.code == 'Free' ? null : limits.balance
                  "
                  v-model="released.current"
                  :rules="[
                    (v) => !!v || 'Обязательное поле',
                    (v) => v >= 1 || 'Не меньше одного литра',
                    (v) => v <= 2000 || 'Не более 2000 литров',
                    (v) =>
                      /^\d*.\d{1}$/.test(v) ||
                      /^\d*$/.test(v) ||
                      'Не более одного знака после запятой',
                  ]"
                  :label="$t('sale.release.label')"
                  :prefix="$t('sale.release.prefix')"
                  :suffix="$t('sale.release.suffix')"
                  hide-details="auto"
                  class="mt-5"
                  @keyup="setDecimalPlace"
                >
                </v-text-field>
                <v-row dense>
                  <v-col cols="12">
                    <div>
                      <p>
                        {{ $t("sale.release.byLimit") }}:
                        {{ released.byLimit }}
                      </p>
                      <p>
                        {{ $t("sale.release.overLimit") }}:
                        {{ released.overLimit }}
                      </p>
                    </div>
                  </v-col>
                </v-row>
              </div>
            </v-flex>
            <v-row justify="space-between" class="mt-2 pr-1">
              <v-btn
                text
                class="mt-3"
                @click="confirmSale"
                :disabled="released.current < 0.1 ? true : false"
              >
                {{ $t("sale.sale") }}
              </v-btn>
              <v-btn text class="mt-3" @click="closeSaleCard">
                {{ $t("sale.cancel") }}
              </v-btn>
            </v-row>
          </div>
        </v-timeline-item>
      </v-timeline>
    </v-card-text>
  </v-card>
</template>

<script>
export default {
  data() {
    return {
      selectedOption: '',
      options: ['ГРНЗ', 'СРТС', 'ПАРКУР']
    }
  }
}
</script>

<script>
export default
data(){
  return{
    selectedOption:'',
    options:['ГРНЗ/ИИН','ГРНХ/СТРС','ПАРКУР']
  }
}

</script>
<script>
import { get } from "vuex-pathify";
import { consumerDocumentsRules } from "@/util/rules";
import { mapActions } from "vuex";
import i18n from "@/i18n";

export default {
  name: "SaleCard",
  components: {},
  computed: {
    getSaleCategoryList: get("sales/getSaleCategoryList"),
    getProductCategoryList: get("sales/getProductCategoryList"),
    documentCategoryList: get("sales/getDocumentCategoryList"),
    getConsumerBalance: get("sales/getConsumerBalance"),
    releasedCurrent: function () {
      return this.released.current;
    },
    changeLocale: function () {
      return this.$i18n.locale;
    },
    changeSaleCategory: function () {
      return this.saleCategory.select;
    },
    cargoCarDocumentPackLabel: function () {
      return this.switchCargoCarDocumentPack
        ? i18n.t("document['ГРНЗ']") + "/" + i18n.t("document['СРТС']")
        : i18n.t("document['ИИН']") + "/" + i18n.t("document['ГРНЗ']");
    },
  },
  props: {
    saleId: {
      type: Number,
    },
    started: {
      type: Number,
    },
  },
  watch: {
    changeSaleCategory: function () {
      this.released.current = 0;
      this.released.byLimit = 0;
      this.released.overLimit = 0;
      this.limits.daily = 0;
      this.limits.balance = 0;
      this.ConsumerBalanceRequest.Type = -1;
      this.ConsumerBalanceRequest.ProductId = -1;
      this.ConsumerBalanceRequest.ConsumerDocuments = [];
    },
    releasedCurrent: function (val) {
      if (this.saleCategory.select.code == "Free") {
        this.released.overLimit = parseFloat(parseFloat(val).toFixed(1));
      } else {
        if (val > this.limits.balance) {
          this.released.overLimit = parseFloat(
            (val - this.limits.balance).toFixed(1)
          );
          this.released.byLimit = this.limits.balance;
        } else {
          this.released.overLimit = 0;
          this.released.byLimit = parseFloat(val.toFixed(1));
        }
      }
    },
    changeLocale: function () {
      this.translateSaleCategoryNames();
    },
  },
  data: () => ({
    maxAmount: 2000,
    smsCode: "",
    isFormValid: false,
    requestConsumerBalanceButtonDisabled: false,
    switchCargoCarDocumentPack: false, // false - 'ГРНЗ/СРТС';  true - 'ИИН/ГРНЗ'
    steps: {
      "Шаг 1 - Выбор параметров отпуска": { show: true },
      "Шаг 2 - Запрос баланса на основании документов": { show: true },
      "Шаг 3 - Отпуск нефтепродуктов": { show: false },
    },

    saleCategory: {
      defaultCode: "ForCargoCar",
      items: [],
      select: null,
    },
    productCategory: {
      // defaultName: "ДТ-Л",
      defaultName: "ДТ",
      items: [],
      select: null,
    },
    ConsumerBalanceRequest: {
      Type: -1,
      ProductId: -1,
      ConsumerDocuments: [],
    },
    limits: {
      daily: 0,
      balance: 0,
    },
    released: {
      current: 0,
      byLimit: 0,
      overLimit: 0,
    },
    document: {
      БИН: {
        value: "",
        max: 12,
        label: () => i18n.t("document['БИН']"), //"БИН",
        rules: [consumerDocumentsRules["БИН"]],
      },
      ИИН: {
        value: "",
        max: 12,
        label: () => i18n.t("document['ИИН']"), //"ИИН",
        rules: [
          consumerDocumentsRules["Обязательное поле"],
          consumerDocumentsRules["ИИН"],
        ],
      },
      ВУ: {
        value: "",
        max: 8,
        label: () => i18n.t("document['ВУ']"), //"ВУ",
        rules: [
          consumerDocumentsRules["Обязательное поле"],
          consumerDocumentsRules["ВУ"],
        ],
      },
      ГРНЗ: {

      data: () => ({
       efaultSelection: '',
       items: [],
       }),
        value: "",
        max: 6,
        label: () => i18n.t("document['ГРНЗ']"), //"ГРНЗ",
        rules: [
          consumerDocumentsRules["Обязательное поле"],
          consumerDocumentsRules["ГРНЗ"],
        ],
      },
      СРТС: {
        value: "",
        max: 10,
        label: () => i18n.t("document['СРТС']"), //"СРТС",
        rules: [
          consumerDocumentsRules["Обязательное поле"],
          consumerDocumentsRules["СРТС"],
        ],
      },
      ТК: {
        value: "",
        max: 8,
        label: () => i18n.t("document['ТК']"),
        rules: [
          consumerDocumentsRules["Обязательное поле"],
          consumerDocumentsRules["ТК"],
        ],
      },
      ОК: {
        value: "",
        max: 6,
        label: () => i18n.t("document['ОК']"),
        rules: [
          consumerDocumentsRules["Обязательное поле"],
          consumerDocumentsRules["ОК"],
        ],
      },
    },
  }),
  methods: {
    ...mapActions({
      fetchConsumerBalance: "sales/fetchConsumerBalance",
      sendSaleRequest: "sales/sendSaleRequest",
    }),
    translateSaleCategoryNames() {
      this.saleCategory.items = this.getSaleCategoryList.map((item) => ({
        id: item.id,
        code: item.code,
        name: item.name(),
      }));
    },
    initialize() {
      // инициализация списка категорий продаж - зашито в стор
      this.translateSaleCategoryNames();
      this.saleCategory.select = this.saleCategory.items.find(
        (item) => item.code == this.saleCategory.defaultCode
      );

      // инициализация списка категорий нефтепродукта - из прайс-листа с backend-а
      this.productCategory.items = this.getProductCategoryList;
      this.productCategory.select = this.productCategory.items.find(
        (item) => item.name == this.productCategory.defaultName
      );
      // console.log(this.documentCategoryList);
      //  documentCategoryList - приходящий с backend-а справочник типов документов
      // заполнение локальных типов документов
      // Новый вариант инициализации документов
      Object.keys(this.document).forEach((key) => {
        Object.assign(
          this.document[key],
          this.documentCategoryList.find((doc) => doc.name == key)
        );
      });

      // Старый вариант инициализации документов
      // Object.assign(
      //   this.document["БИН"],
      //   this.documentCategoryList.find((doc) => doc.name == "БИН")
      // );
      // Object.assign(
      //   this.document["ИИН"],
      //   this.documentCategoryList.find((doc) => doc.name == "ИИН")
      // );
      // Object.assign(
      //   this.document["ВУ"],
      //   this.documentCategoryList.find((doc) => doc.name == "ВУ")
      // );
      // Object.assign(
      //   this.document["ГРНЗ"],
      //   this.documentCategoryList.find((doc) => doc.name == "ГРНЗ")
      // );
      // Object.assign(
      //   this.document["СРТС"],
      //   this.documentCategoryList.find((doc) => doc.name == "СРТС")
      // );
    },
    closeSaleCard() {
      this.$emit("cancel-sale", this.saleId);
    },
    requestConsumerBalance() {
      this.requestConsumerBalanceButtonDisabled = true;
      this.ConsumerBalanceRequest.Type = this.saleCategory.select.id;
      this.ConsumerBalanceRequest.ProductId = this.productCategory.select.id;

      // Новый вариант
      const documentPack =
        this.saleCategory.select.code == "ByDriverLicense"
          ? ["ИИН", "ВУ"]
          : this.saleCategory.select.code == "ForFuelCard" // топливная карта Жанармай - код генерируется приложением
          ? ["ТК"]
          : this.saleCategory.select.code == "ForOnTimeCode" // одноразовый код
          ? ["ОК"]
          : this.saleCategory.select.code == "ForCargoCar"
          ? this.switchCargoCarDocumentPack
            ? ["ГРНЗ", "СРТС"]
            : ["ИИН", "ГРНЗ", "БИН"]
          : [];

      documentPack?.forEach((docType) => {
        this.ConsumerBalanceRequest.ConsumerDocuments.push({
          DocumentId: this.document[docType].id,
          DocumentValue: this.document[docType].value,
        });
      });

      // Старый вариант
      // if (this.saleCategory.select.code == "ByDriverLicense") {
      //   this.ConsumerBalanceRequest.ConsumerDocuments.push(
      //     {
      //       DocumentId: this.document["ИИН"].id,
      //       DocumentValue: this.document["ИИН"].value,
      //     },
      //     {
      //       DocumentId: this.document["ВУ"].id,
      //       DocumentValue: this.document["ВУ"].value,
      //     }
      //   );
      // }
      // if (this.saleCategory.select.code == "ForCargoCar") {
      //   this.switchCargoCarDocumentPack
      //     ? this.ConsumerBalanceRequest.ConsumerDocuments.push(
      //         {
      //           DocumentId: this.document["ГРНЗ"].id,
      //           DocumentValue: this.document["ГРНЗ"].value,
      //         },
      //         {
      //           DocumentId: this.document["СРТС"].id,
      //           DocumentValue: this.document["СРТС"].value,
      //         }
      //       )
      //     : this.ConsumerBalanceRequest.ConsumerDocuments.push(
      //         {
      //           DocumentId: this.document["ИИН"].id,
      //           DocumentValue: this.document["ИИН"].value,
      //         },
      //         {
      //           DocumentId: this.document["ГРНЗ"].id,
      //           DocumentValue: this.document["ГРНЗ"].value,
      //         },
      //         {
      //           DocumentId: this.document["БИН"].id,
      //           DocumentValue: this.document["БИН"].value,
      //         }
      //       );
      // }

      this.fetchConsumerBalance(this.ConsumerBalanceRequest).then(() => {
        if (this.getConsumerBalance != null) {
          if (this.getConsumerBalance.errorMessage !== null) {
            alert(
              `При проверке документов произошла ошибка! \nПерепроверьте документы либо отмените продажу. \n${this.getConsumerBalance.errorMessage}`
            );
            this.ConsumerBalanceRequest.ConsumerDocuments = [];
            this.requestConsumerBalanceButtonDisabled = false;
          } else {
            this.steps[
              "Шаг 2 - Запрос баланса на основании документов"
            ].show = false;
            this.steps["Шаг 3 - Отпуск нефтепродуктов"].show = true;
            this.limits.daily = this.getConsumerBalance.dailyLimit;
            this.limits.balance = this.getConsumerBalance.balance;
            this.limits.verificationToken =
              this.getConsumerBalance.verificationToken;
          }
        }
      });

      // test
      // this.steps["Шаг 2 - Запрос баланса на основании документов"].show = false;
      // this.steps["Шаг 3 - Отпуск нефтепродуктов"].show = true;
      // this.limits.daily = 100;
      // this.limits.balance = 91;
      // this.limits.verificationToken = "100500";
    },
    confirmSale() {
      if (
        confirm(
          `Вы подтверждаете отпуск ${
            this.released.byLimit + this.released.overLimit
          } литров?`
        )
      ) {
        const SaleRequest = {
          ProductId: this.productCategory.select.id,
          DailyLimit: this.limits.daily,
          Balance: this.limits.balance,
          ReleasedByLimit: this.released.byLimit.toFixed(1),
          ReleasedOverLimit: this.released.overLimit.toFixed(1),
          Description: "",
          PosId: this.saleId,
          ConsumerDocuments: [...this.ConsumerBalanceRequest.ConsumerDocuments],
          Type: this.saleCategory.select.id,
          SmsCode: this.smsCode,
          StartDate: new Date(this.started).toISOString(),
          CloseDate: new Date().toISOString(),
          VerificationToken: this.limits.verificationToken,
        };
        if (this.saleCategory.select.code == "Free") {
          SaleRequest.DailyLimit = 0;
          SaleRequest.Balance = 0;
          SaleRequest.VerificationToken = "";
          SaleRequest.ConsumerDocuments = null;
          SaleRequest.Type = 0;
        }
        // console.log(SaleRequest);
        this.sendSaleRequest(SaleRequest);
        this.closeSaleCard();
      } else {
        this.smsCode = "";
        this.released.current = 0;
      }
    },
    setDecimalPlace() {
      const current = parseFloat(parseFloat(this.released.current).toFixed(1));
      Number.isNaN(current)
        ? (this.released.current = 0)
        : (this.released.current =
            current <= this.maxAmount ? current : this.maxAmount);
    },
  },
  created() {
    this.initialize();
  },
};
</script>
