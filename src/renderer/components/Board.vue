<template>
  <div class="tab-content">

    <div>
      <Dropdown placement="bottom-start" style="margin-bottom:10px;" @on-click="addNewHub">
        <Button type="ghost">
            {{$t("m.newHub.button")}}
            <Icon type="arrow-down-b"></Icon>
        </Button>
        <DropdownMenu slot="list">
            <DropdownItem name='new-hub-item'>{{$t("m.newHub.item")}}</DropdownItem>
            <DropdownItem name='new-hub-note'>{{$t("m.newHub.note")}}</DropdownItem>
            <DropdownItem name='new-hub-todo'>{{$t("m.newHub.todo")}}</DropdownItem>
        </DropdownMenu>
      </Dropdown>
      <Tooltip placement="bottom-end" style="float:right; margin-right:5px;">
        <Button type="text" shape="circle" size="small" icon="navicon"
                @click="sortHubModal=true;"
                :disabled="isBoardEmpty"></Button>
        <div slot="content">
          {{$t("m.board.sortTip")}}
        </div>
      </Tooltip>
      <Tooltip placement="bottom-end" style="float:right; margin-right:8px;">
        <Button type="text" shape="circle" size="small" icon="pin" @click="pinBoard"></Button>
        <div slot="content">
          {{$t("m.board.pinTip")}}
        </div>
      </Tooltip>
    </div>
    
    <div v-if="isBoardEmpty" class="info">
      <Alert show-icon>{{$t("m.info.noHubs")}}</Alert>
    </div>
    
    <div v-for="hub in hubs" class="hub" :key="hub.id">
      <hub v-if="hub.type=='item'"
        :name="hub.id"
        :label="hub.label"
        :boardId="boardId"
        :hubId="hub.id"
        @refreshHub="fetchBoardItems"
        ></hub>
        
      <hub-note v-if="hub.type=='note'"
        :name="hub.id"
        :label="hub.label"
        :boardId="boardId"
        :hubId="hub.id"
        @refreshHub="fetchBoardItems"></hub-note>

      <hub-todo v-if="hub.type=='todo'"
        :name="hub.id"
        :label="hub.label"
        :boardId="boardId"
        :hubId="hub.id"
        @refreshHub="fetchBoardItems"></hub-todo>      
    </div>

    <sort-hub-modal :sortHubModal="sortHubModal"
                    :hubs="hubs"
                    @closeSortHubModal="sortHubModal=false;"
                    @submitSortHubs="submitSortHubs">
    </sort-hub-modal>

  </div>
</template>

<script>
import Hub from "./Hub.vue";
import HubNote from "./HubNote.vue";
import HubTodo from "./HubTodo.vue";
import SortHubModal from "./SortHubModal.vue";

import boardsStore from "../store/modules/boardsStore";
import { shell } from "electron";
const app = require("electron").remote.app;

export default {
  props: ["boardId", "label", "selectedTab"],

  components: {
    Hub,
    HubNote,
    HubTodo,
    SortHubModal
  },

  data() {
    return {
      hubs: [],
      sortHubModal: false
    };
  },

  computed: {
    isBoardEmpty() {
      return !this.hubs.length;
    }
  },

  methods: {
    fetchBoardItems() {
      this.hubs = boardsStore.getHubs(this.boardId);
    },
    addNewHub: function(name) {
      if (name == "new-hub-item") {
        boardsStore.addHubToEnd(
          this.boardId,
          "item",
          this.$i18n.t("m.newHub.item")
        );
        this.fetchBoardItems();
      } else if (name == "new-hub-note") {
        boardsStore.addHubToEnd(
          this.boardId,
          "note",
          this.$i18n.t("m.newHub.note")
        );
        this.fetchBoardItems();
      } else if (name == "new-hub-todo") {
        boardsStore.addHubToEnd(
          this.boardId,
          "todo",
          this.$i18n.t("m.newHub.todo")
        );
        this.fetchBoardItems();
      }
    },
    pinBoard() {
      let homedir = app.getPath("desktop");
      let exePath = require("process").execPath;
      let shortcut = {
        target: exePath,
        cwd: require("path").dirname(exePath),
        args: `--board ${this.boardId}`
      };

      shell.writeShortcutLink(
        require("path").join(homedir, this.label + ".lnk"),
        shortcut
      );

      this.$Message.success(this.$i18n.t("m.board.pinOK"));
    },
    submitSortHubs(hubs) {
      this.sortHubModal = false;
      boardsStore.saveHubsArray(this.boardId, hubs);
      this.fetchBoardItems();

      this.$Message.success(this.$i18n.t("m.modal.sort.success"));
    }
  },

  created() {
    this.fetchBoardItems();
  }
};
</script>

<style>
.tab-content .hub {
  margin-bottom: 10px;
  padding: 0px 1px;
}
.tab-content {
  min-height: 300px;
}
</style>