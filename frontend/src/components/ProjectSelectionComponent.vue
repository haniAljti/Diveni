<template>
  <div>
    <b-input-group>
      <b-input-group-prepend>
        <b-input-group-text><BIconSearch id="searchIcon"></BIconSearch></b-input-group-text>
      </b-input-group-prepend>
      <b-input
        id="search"
        v-model="input"
        type="text"
        :placeholder="
          t(
            'session.prepare.step.selection.mode.description.withIssueTracker.placeholder.searchProjects'
          )
        "
        @input="
          showResult = true;
          filterProjects();
        "
        @click="
          showResult = true;
          filterProjects();
        "
        @blur="input === '' ? (showResult = false) : (showResult = true)"
      />
    </b-input-group>
    <ul v-if="showResult" class="vue-autocomplete-input-tag-items">
      <li
        v-for="name in projectNamesList"
        :key="name"
        class="vue-autocomplete-input-tag-item"
        @click="selectProject(name)"
      >
        {{ name }}
      </li>
    </ul>

    <div class="mt-3">
      {{ t("session.prepare.step.selection.mode.description.withIssueTracker.selectedProject") }}
      <strong>{{ aCorrectProject ? selected : "-" }}</strong>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import apiService from "@/services/api.service";
import Project from "../model/Project";
import { useDiveniStore } from "@/store";
import { useI18n } from "vue-i18n";

export default defineComponent({
  name: "ProjectSelectionComponent",

  setup() {
    const store = useDiveniStore();
    const { t } = useI18n();
    return { store, t };
  },
  data() {
    return {
      selected: "",
      projectArray: [] as Array<Project>,
      projectNamesList: [] as Array<string>,
      input: "",
      aCorrectProject: false,
      showResult: false,
    };
  },
  computed: {
    projects(): Array<Project> {
      return this.store.projects;
    },
  },

  methods: {
    async getUserStories() {
      this.aCorrectProject = false;
      const selectedProject = this.projects.find((p) => p.name === this.selected);
      if (selectedProject) {
        this.aCorrectProject = true;
        console.log(`Selected: ${selectedProject}`);
        this.store.setSelectedProject(selectedProject);
        console.log(`Selected Project: ${this.selected}`);
        const response = await apiService.getUserStoriesFromProject(this.selected);
        this.store.setUserStories({ stories: response });
      }
      this.showResult = false;
    },
    filterProjects: function () {
      if (this.input === "") {
        this.projectNamesList = this.getProjectNames();
        return;
      }
      if (this.input !== "") {
        let filteredProjects: string[] = [];
        this.getProjectNames().forEach((name) => {
          if (name.toLowerCase().includes(this.input.toLowerCase())) {
            filteredProjects.push(name);
          }
        });
        this.projectNamesList = filteredProjects;
      }
    },
    getProjectNames(): Array<string> {
      return this.projects.map((p) => p.name);
    },
    selectProject(name: string) {
      this.projects.forEach((project) => {
        if (project.name === name) {
          this.selected = project.name;
        }
      });
      this.input = this.selected;
      this.getUserStories();
    },
  },
});
</script>

<style lang="scss">
input {
  width: 100%;
  border: 1px solid #ccc;
  color: #666;
  border-radius: 10px;
  outline: none;
  padding: 9px 14px;
  box-sizing: border-box;
  font-size: 14px;
}
.vue-autocomplete-input-tag-items {
  border: 1px solid #ccc;
  max-height: 200px;
  margin-top: 8px;
  width: 100%;
  background-color: white;
  border-radius: 8px;
  overflow: auto;
}
.vue-autocomplete-input-tag-item {
  padding: 6px 16px;
  color: #4a4a4a;
  max-width: 100%;
  cursor: pointer;
  text-align: left;
  font-size: 14px;
}
.vue-autocomplete-input-tag-item:hover {
  background-color: #e8e8e8;
}
</style>
