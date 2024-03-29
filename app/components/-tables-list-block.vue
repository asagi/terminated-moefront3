<template>
  <div class="tables-block">
    <div
      :class="{
        open: isOpened,
        new: mode == Const.MODE_NEW,
        live: mode == Const.MODE_LIVE,
        closed: mode == Const.MODE_CLOSED
      }"
      class="tables"
    >
      <h2 @click="isOpened = !isOpened" :class="{ open: isOpened }">
        <template v-if="mode == Const.MODE_NEW">
          募集中の卓の一覧
        </template>
        <template v-if="mode == Const.MODE_LIVE">
          進行中の卓の一覧
        </template>
        <template v-if="mode == Const.MODE_CLOSED">
          終了した卓の一覧
        </template>
      </h2>
      <transition
        @before-enter="beforeEnter"
        @enter="enter"
        @before-leave="beforeLeave"
        @leave="leave"
      >
        <div v-if="isOpened" class="talbe-wrapper">
          <div v-if="tables.length === 0" class="empty-message">
            <template v-if="mode == Const.MODE_NEW">
              現在募集中の卓はありません。
            </template>
            <template v-if="mode == Const.MODE_LIVE">
              現在進行中の卓はありません。
            </template>
            <template v-if="mode == Const.MODE_CLOSED">
              終了した卓はありません。
            </template>
          </div>
          <table v-if="tables.length > 0">
            <tbody>
              <tr v-for="table in tables" v-bind:key="table.id">
                <template v-if="mode == Const.MODE_NEW">
                  <th data-label="卓主" class="with-label new">
                    {{ table.owner.name }}
                    <span class="twttier-account">
                      <a :href="table.owner.url" target="_blank">
                        {{ '@' + table.owner.nickname }}
                      </a>
                    </span>
                  </th>
                  <td data-label="開始日時" class="with-label">
                    {{ $formatPeriod(table.period) }}
                  </td>
                  <td data-label="フェイス" class="with-label">
                    {{ $getFaceType(table.regulation.face_type) }}
                  </td>
                  <td data-label="外交期間" class="with-label half">
                    {{ $getDuration(table.regulation.duration) }}
                  </td>
                  <td data-label="更新種別" class="with-label half">
                    {{ $getPeriodRule(table.regulation.period_rule) }}
                  </td>
                  <td data-label="掛け持ち" class="with-label half">
                    {{ $getJugglingState(table.regulation.juggling) }}
                  </td>
                  <td data-label="鍵" class="with-label half">
                    {{ $getPrivateState(table.private) }}
                  </td>
                </template>

                <template v-if="mode == Const.MODE_LIVE">
                  <th
                    :class="{
                      girl: table.regulation.face_type == 'girl',
                      flag: table.regulation.face_type == 'flag'
                    }"
                  >
                    <span class="turn-number">
                      {{ $getTableNumber(table.number) }}
                      {{ $getFaceType(table.regulation.face_type) }}
                      :
                    </span>
                    {{ $getTurn(table.turn, table.phase) }}
                  </th>

                  <td data-label="卓主" class="with-label">
                    {{ table.owner.name }}
                    <span class="twttier-account">
                      <a :href="table.owner.url" target="_blank">
                        {{ '@' + table.owner.nickname }}
                      </a>
                    </span>
                  </td>
                  <td data-label="次回更新" class="with-label">
                    {{ $formatPeriod(table.period) }}
                  </td>
                  <td data-label="外交期間" class="with-label half">
                    {{ $getDuration(table.regulation.duration) }}
                  </td>
                  <td data-label="更新種別" class="with-label half">
                    {{ $getPeriodRule(table.regulation.period_rule) }}
                  </td>
                  <td data-label="掛け持ち" class="with-label">
                    {{ $getJugglingState(table.regulation.juggling) }}
                  </td>
                </template>

                <template v-if="mode == Const.MODE_CLOSED">
                  <th
                    :class="{
                      girl: table.regulation.face_type == 'girl',
                      flag: table.regulation.face_type == 'flag'
                    }"
                  >
                    <span class="turn-number">
                      {{ $getTableNumber(table.number) }}
                      {{ $getFaceType(table.regulation.face_type) }}
                      :
                    </span>
                    {{ $getFinishTurn(table.turn, table.phase) }}
                  </th>

                  <td data-label="卓主" class="with-label">
                    {{ table.owner.name }}
                    <span class="twttier-account">
                      <a :href="table.owner.url" target="_blank">
                        {{ '@' + table.owner.nickname }}
                      </a>
                    </span>
                  </td>
                  <td data-label="終了日時" class="with-label">
                    {{ $formatPeriod(table.period) }}
                  </td>
                  <td data-label="外交期間" class="with-label half">
                    {{ $getDuration(table.regulation.duration) }}
                  </td>
                  <td data-label="更新種別" class="with-label half">
                    {{ $getPeriodRule(table.regulation.period_rule) }}
                  </td>
                </template>

                <td class="button">
                  <nuxt-link
                    v-if="table.status == 'created' || table.status == 'ready'"
                    :to="'/entries/' + table.id"
                    class="btn btn-blue"
                  >
                    表示する
                  </nuxt-link>
                  <nuxt-link
                    v-else
                    :to="'/tables/' + table.number"
                    class="btn btn-blue"
                  >
                    表示する
                  </nuxt-link>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </transition>
    </div>
  </div>
</template>

<script>
import debounce from 'lodash/debounce'

export default {
  props: {
    mode: {
      type: Number,
      default: () => {
        return 1
      }
    },
    list: {
      type: Array,
      default: () => {
        return []
      }
    }
  },
  data() {
    return {
      Const: {
        MODE_NEW: 0,
        MODE_LIVE: 1,
        MODE_CLOSED: 2
      },
      tables: [],
      isOpened: false,
      width: 0
    }
  },
  computed: {
    isSP: function() {
      return this.width < 1024
    }
  },
  created: function() {
    this.handleResize = debounce(this.resize, 100)
  },
  mounted: function() {
    window.addEventListener('resize', this.handleResize)
    switch (this.mode) {
      case this.Const.MODE_NEW:
        this.tables = this.list.filter((value, index, array) => {
          if (value.turn !== 0) return false
          if (value.status === 'created') return true
          if (value.status === 'ready') return true
          return false
        })
        break
      case this.Const.MODE_LIVE:
        this.tables = this.list.filter((value, index, array) => {
          if (value.status === 'started') return true
          if (value.status === 'draw') return true
          if (value.status === 'solo') return true
          return false
        })
        break
      case this.Const.MODE_CLOSED:
        this.tables = this.list.filter((value, index, array) => {
          if (value.status === 'closed') return true
          return false
        })
        break
      default:
        throw new Error()
    }
  },
  beforeDestroy: function() {
    window.removeEventListener('resize', this.handleResize)
  },
  methods: {
    resize: function() {
      this.width = window.innerWidth
    },
    beforeEnter: el => {
      el.style.height = '0'
    },
    enter: el => {
      el.style.height = el.scrollHeight + 'px'
    },
    beforeLeave: el => {
      el.style.height = el.scrollHeight + 'px'
    },
    leave: el => {
      el.style.height = '0'
    }
  }
}
</script>

<style lang="postcss" scoped>
.tables-block {
  @apply my-0;
  margin-top: -1px;

  & .tables {
    @apply my-0;

    & h2 {
      @apply py-3 my-0;
      @apply bg-gray-100;
      @apply border-solid border-t border-b border-gray-400;
      @apply cursor-pointer;

      &::before {
        @apply inline-block relative mr-3;
        content: '';
        width: 12px;
        height: 12px;
        border-top: solid 2px #000;
        border-right: solid 2px #000;
        transform: rotate(135deg);
      }
      &.open {
        &::before {
          transform: rotate(-45deg);
        }
      }
    }

    & .talbe-wrapper {
      @apply my-0 py-2;
      overflow-y: hidden;
      transition: height 0.2s ease;

      & .empty-message {
        @apply mt-1 py-4 px-4 mx-4;
        @apply bg-gray-400 rounded-lg;
        @apply text-center;
      }

      & table {
        @apply w-full mx-0 my-auto;
        @apply border-separate border-0;
        @apply leading-normal;

        & thead {
          display: none;
        }

        & tbody {
          & tr {
            @apply block mx-2 mb-2 p-2;
            @apply rounded shadow;
            @apply border-solid border border-gray-400;
            @apply bg-white;

            & th {
              @apply block overflow-hidden;
              @apply rounded;

              &.new {
                /* mode == 0 */
                @apply text-left px-2 py-2;
                @apply relative pl-16;
                @apply font-normal;
                @apply bg-indigo-200;
              }

              &.girl {
                /* mode == 1 or 2*/
                @apply text-left px-4 py-2;
                @apply bg-pink-200;
              }
              &.flag {
                /* mode == 1 or 2*/
                @apply text-left px-4 py-2;
                @apply bg-orange-200;
              }

              & span.table-number {
                /* mode == 1 or 2*/
                @apply font-bold;
              }
            }

            & td {
              @apply block;
              @apply text-left px-2 py-2;
              @apply relative pl-24;
              @apply font-normal text-sm;
              @apply border-b;

              &.half {
                @apply w-1/2 -mx-1;
                @apply inline-block;
              }

              &.button {
                @apply px-0 pt-4;
                @apply text-center;
              }

              & .twttier-account {
                @apply text-xs;
              }
            }

            & th.with-label::before {
              @apply font-bold;
              @apply absolute;
              content: attr(data-label) ' : ';
              left: 10px;
            }

            & td.with-label::before {
              @apply font-bold;
              @apply absolute inline-block w-20 text-right;
              content: attr(data-label) ' : ';
              left: 0;
            }
            & td:last-child {
              @apply border-b-0;
            }
          }
        }
      }
    }
  }
}

@screen sm {
  .tables-block {
    & .tables.live {
      & h2 {
        @apply mt-10;
      }
    }

    & .tables.closed {
      & h2 {
        @apply mt-10;
      }
    }
  }
}
</style>
