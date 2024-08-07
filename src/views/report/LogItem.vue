<template>
  <div
    class="vx-row overflow-hidden"
    :class="{ expand: expand, 'no-expand': !expand }"
  >
    <div class="vx-col w-full cursor-pointer" @click="expand = !expand">
      <div class="sm:flex justify-between items-center mb-1">
        <div class="flex">
          <h3 class="karla-bold" style="font-size: 16px">
            {{ template(log.templateID).content.templateTitle | capitalize }}
            {{ scheduleLocation }}
          </h3>
          <vs-icon
            class="font-bold ml-4 expand-icon"
            :class="{ on: expand, off: !expand }"
            color="black"
            icon-pack="feather"
            size="20px"
            icon="icon-chevron-down"
          />
        </div>
        <div class="text-right" v-if="log.updated_at">
          <span style="font-size: 14px; font-weight: bold; color: #1e1c26">{{
            log.updated_at.toDate() | moment("DD MMM YYYY")
          }}</span>
        </div>
      </div>
      <span class="text-grey text-sm">{{ answerState(log) }}</span>
    </div>
    <vs-collapse
      class="w-full logContent"
      v-for="(page, pIndex) in log.logs"
      :key="'q' + pIndex"
    >
      <vs-collapse-item :open="pIndex == 0">
        <div slot="header">
          <!-- class="border pb-2 border-solid border-t-0 border-t-0 border-r-0 border-l-0 d-theme-border-grey-light" -->
          <h4
            class="overflow-hidden mr-12"
            style="font-size: 14px; font-weight: bold; color: #1e1c26"
          >
            {{ page.title | capitalize }}
            <span
              class="text-sm bg-grey rounded-full px-4 py-2 ml-4 pageState"
              >{{ pageState(page) }}</span
            >
          </h4>
        </div>
        <div>
          <table
            v-for="(question, qIndex) in page.questions"
            :key="'q' + qIndex"
            class="logTable mb-4 rounded-lg w-full border border-solid d-theme-border-grey-light"
          >
            <tr class>
              <td colspan="2">
                <h4
                  class="p-4"
                  style="
                    border-radius: 0.5rem 0.5rem 0 0;
                    background-color: rgba(113, 102, 237, 0.08);
                    font-size: 14px;
                    font-weight: 500;
                  "
                >
                  {{ question.title }}
                </h4>
              </td>
            </tr>
            <template v-for="(answer, aIndex) in question.answers">
              <tr
                :key="'a' + aIndex"
                v-if="
                  getType(answer.ref.type.id) !== undefined &&
                  answer.ref.isLogicQuestion != true
                "
              >
                <template>
                  <td
                    width="30%"
                    class="p-2 py-3 border border-solid border-r-0 border-l-0 d-theme-border-grey-light border-b-0 text-large"
                  >
                    <h5
                      style="
                        display: inline;
                        font-size: 12px;
                        font-weight: 500;
                        color: #1e1c26;
                        opacity: 0.78;
                      "
                    >
                      {{ answer.ref.title }}
                      <template
                        v-if="
                          getType(answer.ref.type.id).content == 'temperature'
                        "
                      >
                        <span
                          v-if="answer.ref.type.tempUnit"
                          class="text-success"
                          >({{ answer.ref.type.tempUnit }})&nbsp;</span
                        >
                        <span v-else class="text-success">(℃)&nbsp;</span>
                      </template>
                      <span v-if="answer.ref.mandatory" class="text-danger"
                        >*</span
                      >
                    </h5>
                  </td>
                  <td
                    class="p-2 py-4 border border-solid d-theme-border-grey-light border-b-0 border-r-0"
                  >
                    <span class="text-danger" v-if="!answer.loged">---</span>
                    <span
                      v-else-if="
                        getType(answer.ref.type.id).content ==
                          'automatic date and time stamp' ||
                        getType(answer.ref.type.id).content == 'date'
                      "
                      >{{
                        convDate(answer.value)
                          | moment("DD MMMM ,YYYY @ HH[h]mm")
                      }}</span
                    >
                    <template
                      v-else-if="
                        getType(answer.ref.type.id).content == 'date time'
                      "
                    >
                      <span v-if="answer.ref.type.dateType == 'Date'">{{
                        answer.value | moment("DD MMMM ,YYYY")
                      }}</span>
                      <span v-else-if="answer.ref.type.dateType == 'Time'">{{
                        answer.value | moment("HH:mm")
                      }}</span>
                      <span
                        v-else-if="answer.ref.type.dateType == 'Date & Time'"
                        >{{
                          answer.value | moment("DD MMMM ,YYYY @ HH[h]mm")
                        }}</span
                      >
                    </template>
                    <star-rating
                      v-else-if="getType(answer.ref.type.id).content == 'star'"
                      :star-size="25"
                      :read-only="true"
                      :increment="0.1"
                      :fixed-points="1"
                      :rating="answer.value"
                    ></star-rating>
                    <span
                      v-else-if="
                        getType(answer.ref.type.id).content ==
                          'automatic user stamp' &&
                        getUserInfo(answer.value) != undefined
                      "
                      >{{ getUserInfo(answer.value).name | capitalize }}</span
                    >
                    <div
                      v-else-if="
                        getType(answer.ref.type.id).content == 'short answer' ||
                        getType(answer.ref.type.id).content == 'paragraph'
                      "
                      class="text-primary"
                      style="
                        font-style: italic;
                        white-space: pre-wrap;
                        word-break: break-word;
                      "
                    >
                      {{ answer.value }}
                    </div>
                    <vs-checkbox
                      v-else-if="
                        getType(answer.ref.type.id).content == 'checkbox'
                      "
                      disabled="true"
                      :value="answer.value"
                    ></vs-checkbox>
                    <template
                      v-else-if="getType(answer.ref.type.id).content == 'items'"
                    >
                      <span v-if="getItemInfo(answer.value) != undefined">{{
                        getItemInfo(answer.value).name
                      }}</span>
                    </template>
                    <template
                      v-else-if="getType(answer.ref.type.id).content == 'users'"
                    >
                      <span v-if="getUserInfo(answer.value) != undefined">{{
                        getUserInfo(answer.value).name | capitalize
                      }}</span>
                    </template>
                    <template
                      v-else-if="getType(answer.ref.type.id).content == 'teams'"
                    >
                      <span v-if="getTeamInfo(answer.value) != undefined">{{
                        getTeamInfo(answer.value).name | capitalize
                      }}</span>
                    </template>
                    <template
                      v-else-if="
                        getType(answer.ref.type.id).type == 'attachments'
                      "
                    >
                      <a
                        :href="image.url"
                        target="_blank"
                        v-for="(image, imgIndex) in answer.images"
                        :key="'img_' + imgIndex"
                        class="ml-1"
                        style="float: left"
                      >
                        <img
                          :src="image.url"
                          style="max-width: 50px; max-height: 50px"
                          class="rounded-lg"
                        />
                      </a>
                    </template>
                    <div
                      v-else-if="
                        getType(answer.ref.type.id).content == 'instruction'
                      "
                      style="
                        font-size: 12px;
                        color: #86848a;
                        font-weight: normal;
                        white-space: pre-wrap;
                        word-break: break-word;
                      "
                    >
                      {{ answer.ref.type.instruction }}
                    </div>
                    <template
                      v-else-if="
                        getType(answer.ref.type.id).content == 'temperature'
                      "
                    >
                      <span
                        :class="
                          calcTemperatureColor(answer.ref.action, answer.value)
                        "
                        v-if="answer.value !== undefined"
                        >{{ answer.value }}</span
                      >
                    </template>
                    <template
                      v-else-if="
                        getType(answer.ref.type.id).content == 'signature'
                      "
                    >
                      <img
                        :src="answer.value.url"
                        v-if="
                          answer.value.url !== undefined &&
                          answer.value.url != ''
                        "
                      />
                    </template>
                    <template
                      v-else-if="
                        getType(answer.ref.type.id).type == 'closed answers'
                      "
                    >
                      <span
                        :class="[
                          {
                            'text-danger':
                              answer.ref.type.failedAnswer == answer.value,
                          },
                        ]"
                        v-if="getType(answer.ref.type.id).group == 'global'"
                        >{{ $t(answer.value) }}</span
                      >
                      <span
                        :class="[
                          {
                            'text-danger':
                              answer.ref.type.failedAnswer == answer.value,
                          },
                        ]"
                        v-else
                        >{{ answer.value | capitalize }}</span
                      >
                    </template>
                    <span v-else>{{ answer.value }}</span>
                  </td>
                </template>
              </tr>

              <template
                v-for="cAnswer in filteredAnswers(question.answers, answer)"
              >
                <tr
                  v-if="getType(cAnswer.ref.type.id) !== undefined"
                  :key="cAnswer.id"
                >
                  <template>
                    <td
                      width="30%"
                      class="p-2 py-3 border border-solid border-r-0 border-l-0 d-theme-border-grey-light border-b-0 text-large"
                    >
                      <h5
                        style="
                          display: inline;
                          font-size: 12px;
                          font-weight: 500;
                          color: #1e1c26;
                          opacity: 0.78;
                        "
                      >
                        {{ cAnswer.ref.title }}
                        <template
                          v-if="
                            getType(cAnswer.ref.type.id).content ==
                            'temperature'
                          "
                        >
                          <span
                            v-if="cAnswer.ref.type.tempUnit"
                            class="text-success"
                            >({{ cAnswer.ref.type.tempUnit }})&nbsp;</span
                          >
                          <span v-else class="text-success">(℃)&nbsp;</span>
                        </template>
                        <span v-if="cAnswer.ref.mandatory" class="text-danger"
                          >*</span
                        >
                      </h5>
                    </td>
                    <td
                      class="p-2 py-4 border border-solid d-theme-border-grey-light border-b-0 border-r-0"
                    >
                      <span class="text-danger" v-if="!cAnswer.loged">---</span>
                      <span
                        v-else-if="
                          getType(cAnswer.ref.type.id).content ==
                            'automatic date and time stamp' ||
                          getType(cAnswer.ref.type.id).content == 'date'
                        "
                        >{{
                          convDate(cAnswer.value)
                            | moment("DD MMMM ,YYYY @ HH[h]mm")
                        }}</span
                      >
                      <template
                        v-else-if="
                          getType(cAnswer.ref.type.id).content == 'date time'
                        "
                      >
                        <span v-if="cAnswer.ref.type.dateType == 'Date'">{{
                          cAnswer.value | moment("DD MMMM ,YYYY")
                        }}</span>
                        <span v-else-if="cAnswer.ref.type.dateType == 'Time'">{{
                          cAnswer.value | moment("HH:mm")
                        }}</span>
                        <span
                          v-else-if="cAnswer.ref.type.dateType == 'Date & Time'"
                          >{{
                            cAnswer.value | moment("DD MMMM ,YYYY @ HH[h]mm")
                          }}</span
                        >
                      </template>
                      <star-rating
                        v-else-if="
                          getType(cAnswer.ref.type.id).content == 'star'
                        "
                        :star-size="25"
                        :read-only="true"
                        :increment="0.1"
                        :fixed-points="1"
                        :rating="cAnswer.value"
                      ></star-rating>
                      <span
                        v-else-if="
                          getType(cAnswer.ref.type.id).content ==
                            'automatic user stamp' &&
                          getUserInfo(cAnswer.value) != undefined
                        "
                        >{{
                          getUserInfo(cAnswer.value).name | capitalize
                        }}</span
                      >
                      <div
                        v-else-if="
                          getType(cAnswer.ref.type.id).content ==
                            'short answer' ||
                          getType(cAnswer.ref.type.id).content == 'paragraph'
                        "
                        class="text-primary"
                        style="
                          font-style: italic;
                          white-space: pre-wrap;
                          word-break: break-word;
                        "
                      >
                        {{ cAnswer.value }}
                      </div>
                      <vs-checkbox
                        v-else-if="
                          getType(cAnswer.ref.type.id).content == 'checkbox'
                        "
                        disabled="true"
                        :value="cAnswer.value"
                      ></vs-checkbox>
                      <template
                        v-else-if="
                          getType(cAnswer.ref.type.id).content == 'items'
                        "
                      >
                        <span v-if="getItemInfo(cAnswer.value) != undefined">{{
                          getItemInfo(cAnswer.value).name
                        }}</span>
                      </template>
                      <template
                        v-else-if="
                          getType(cAnswer.ref.type.id).content == 'users'
                        "
                      >
                        <span v-if="getUserInfo(cAnswer.value) != undefined">{{
                          getUserInfo(cAnswer.value).name | capitalize
                        }}</span>
                      </template>
                      <template
                        v-else-if="
                          getType(cAnswer.ref.type.id).content == 'teams'
                        "
                      >
                        <span v-if="getTeamInfo(cAnswer.value) != undefined">{{
                          getTeamInfo(cAnswer.value).name | capitalize
                        }}</span>
                      </template>
                      <template
                        v-else-if="
                          getType(cAnswer.ref.type.id).type == 'attachments'
                        "
                      >
                        <a
                          :href="image.url"
                          target="_blank"
                          v-for="(image, imgIndex) in cAnswer.images"
                          :key="'img_' + imgIndex"
                          class="ml-1"
                          style="float: left"
                        >
                          <img
                            :src="image.url"
                            style="max-width: 50px; max-height: 50px"
                            class="rounded-lg"
                          />
                        </a>
                      </template>
                      <div
                        v-else-if="
                          getType(cAnswer.ref.type.id).content == 'instruction'
                        "
                        style="
                          font-size: 12px;
                          color: #86848a;
                          font-weight: normal;
                          white-space: pre-wrap;
                          word-break: break-word;
                        "
                      >
                        {{ cAnswer.ref.type.instruction }}
                      </div>
                      <template
                        v-else-if="
                          getType(cAnswer.ref.type.id).content == 'temperature'
                        "
                      >
                        <span
                          :class="
                            calcTemperatureColor(
                              cAnswer.ref.action,
                              cAnswer.value
                            )
                          "
                          v-if="cAnswer.value !== undefined"
                          >{{ cAnswer.value }}</span
                        >
                      </template>
                      <template
                        v-else-if="
                          getType(cAnswer.ref.type.id).content == 'signature'
                        "
                      >
                        <img
                          :src="cAnswer.value.url"
                          v-if="
                            cAnswer.value.url !== undefined &&
                            cAnswer.value.url != ''
                          "
                        />
                      </template>
                      <template
                        v-else-if="
                          getType(cAnswer.ref.type.id).type == 'closed answers'
                        "
                      >
                        <span
                          :class="[
                            {
                              'text-danger':
                                cAnswer.ref.type.failedAnswer == cAnswer.value,
                            },
                          ]"
                          v-if="getType(cAnswer.ref.type.id).group == 'global'"
                          >{{ $t(cAnswer.value) }}</span
                        >
                        <span
                          :class="[
                            {
                              'text-danger':
                                cAnswer.ref.type.failedAnswer == cAnswer.value,
                            },
                          ]"
                          v-else
                          >{{ cAnswer.value | capitalize }}</span
                        >
                      </template>
                      <span v-else>{{ cAnswer.value }}</span>
                    </td>
                  </template>
                </tr>
              </template>
            </template>
          </table>
        </div>
      </vs-collapse-item>
    </vs-collapse>
  </div>
</template>

<script>
import StarRating from "vue-star-rating";
import ClosedAnswerLogItem from "./ConditionalLogic/ClosedAnswerLogItem";

export default {
  components: {
    StarRating,
    ClosedAnswerLogItem,
  },
  props: {
    log: {
      type: Object,
      required: true,
    },
    order: {
      type: Number,
      required: true,
    },
  },
  data() {
    return {
      expand: false,
    };
  },
  computed: {
    scheduleLocation() {
      return "";
      // if(!this.log.schedule) return '-------- no schedule'
      // else {
      //     var schedule = this.$store.getters['app/getScheduleById'](this.log.schedule || '')
      //     if(!schedule.location) return '----------- no location'
      //     return "---------------" + this.$store.getters['app/getLocationById'](schedule.location[0]).name
      // }
    },
    convDate() {
      return (date) => {
        if (date.seconds === undefined) return date;
        return new Date(date.seconds * 1000);
      };
    },
    getTeamInfo() {
      return (id) => {
        return this.$store.getters["app/getTeamById"](id);
      };
    },
    template() {
      return (id) => {
        return this.$store.getters["app/getTemplateById"](id);
      };
    },
    answerState() {
      return (log) => {
        var all = 0;
        var complate = 0;
        log.logs.map((log) => {
          log.questions.map((question) => {
            question.answers.map((answer) => {
              if (answer.ref.mandatory) {
                all++;
                if (answer.loged) complate++;
              }
            });
          });
        });
        return `(${complate} / ${all})`;
      };
    },
    pageState() {
      return (log) => {
        var all = 0;
        var complate = 0;
        log.questions.map((question) => {
          question.answers.map((answer) => {
            if (answer.ref.mandatory) {
              all++;
              if (answer.loged) complate++;
            }
          });
        });
        return `${complate} / ${all}`;
      };
    },
    getType() {
      return (id) => {
        var type = this.$store.getters["app/getTemplateTypeById"](id);
        return type;
      };
    },
    getUserInfo() {
      return (id) => {
        return this.$store.getters["app/getUserById"](id);
      };
    },
    getItemInfo() {
      return (id) => {
        return this.$store.getters["app/getItemById"](id);
      };
    },
    calcTemperatureColor() {
      return (action, e) => {
        if (action === undefined || action.condition === undefined) return "";
        if (
          (action.condition == "equal" && e == action.content[0]) ||
          (action.condition == "not equal" && e != action.content[0]) ||
          (action.condition == "less than" && e < action.content[0]) ||
          (action.condition == "less than or equal" &&
            e <= action.content[0]) ||
          (action.condition == "greater than" && e > action.content[0]) ||
          (action.condition == "greater than or equal" &&
            e >= action.content[0]) ||
          (action.condition == "between" &&
            e >= action.content[0] &&
            e <= action.content[1])
        ) {
          return "text-" + action.alertType + " font-bold";
        }
      };
    },
  },
  mounted() {},
  beforeMount() {
    if (this.order != 0) this.expand = true;
  },
  methods: {
    filteredAnswers(answers, parent) {
      if (parent.ref.hasCondLogic != true) {
        return;
      }
      // get only answers which is matched to tab condition
      answers = answers.filter((a) => a.ref.parent == parent.ref.id);
      let selectedAnswer = parent.value;
      let matchingTabs = [];
      if (selectedAnswer == "") {
        matchingTabs = this.conditionTabs(parent.ref.id).filter(
          (tab) => tab.condition.symbol == "not_selected"
        );
      } else {
        matchingTabs = this.conditionTabs(parent.ref.id).filter((tab) => {
          if (tab.condition.symbol == "selected") {
            return true;
          } else {
            if (
              tab.condition.key == "is" &&
              tab.answers[0].name == selectedAnswer
            ) {
              return true;
            } else if (
              tab.condition.key == "is_not" &&
              tab.answers[0].name != selectedAnswer
            ) {
              return true;
            } else if (
              tab.condition.key == "is_one_of" &&
              tab.answers.some((answer) => answer.name == selectedAnswer)
            ) {
              return true;
            } else if (
              tab.condition.key == "is_not_one_of" &&
              tab.answers.some((answer) => answer.name == selectedAnswer)
            ) {
              return false;
            } else {
              return false;
            }
          }
        });
      }

      const filtered = answers.filter((answer) => {
        return matchingTabs.some((tab) => {
          return tab.id == answer.ref.tabId;
        });
      });

      return filtered;
    },
    conditionTabs(parentAnswerId) {
      if (
        this.template(this.log.templateID).content.conditionTabs == undefined
      ) {
        return [];
      }

      return this.template(this.log.templateID).content.conditionTabs.filter(
        (tab) => tab.createdByAnswer == parentAnswerId
      );
    },
  },
};
</script>

<style scoped>
.expand-icon {
  -webkit-transition: all 0.25s ease;
  transition: all 0.25s ease;
}
.expand-icon.on {
  -webkit-transform: rotate(180deg);
  transform: rotate(180deg);
}
.expand-icon.off {
  -webkit-transform: rotate(0deg);
  transform: rotate(0deg);
}
.logContent {
  overflow: hidden;
  -webkit-transition: max-height 0.25s ease;
  -moz-transition: max-height 0.25s ease;
  -o-transition: max-height 0.25s ease;
  transition: max-height 0.25s ease;
}
.expand .logContent {
  max-height: 0px;
  padding: 0;
}
.no-expand .logContent {
  max-height: 100000px;
}
.logTable {
  border-spacing: 0px;
}
.pageState {
  display: inline-block;
  position: relative;
}
.pageState::after {
  content: "";
  position: absolute;
  height: 1px;
  border-top: 1px solid #c1c1c1;
  top: 15px;
  width: 1600px;
  margin-left: 2rem;
}
</style>

<style>
#report-create .vs-collapse {
  padding: 0px;
}
#public-report .icon-header.vs-collapse-item--icon-header,
#report-create .icon-header.vs-collapse-item--icon-header,
#report-view .icon-header.vs-collapse-item--icon-header {
  width: 32px;
  border-radius: 50%;
  color: #7367f0;
  height: 32px;
  background-color: #7367f030;
}
</style>

<style >
.open-item .vs-collapse-item--content {
  max-height: unset !important;
}
</style>