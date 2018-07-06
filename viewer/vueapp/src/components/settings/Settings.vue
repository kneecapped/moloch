<template>

  <!-- settings content -->
  <div class="settings-page">

    <!-- sub navbar -->
    <div class="sub-navbar">
      <span class="sub-navbar-title">
        <span class="fa-stack">
          <span class="fa fa-cogs fa-stack-1x"></span>
          <span class="fa fa-square-o fa-stack-2x"></span>
        </span>&nbsp;
        <span>
          Settings
          <span v-if="displayName">
            for {{ displayName }}
          </span>
        </span>
      </span>
      <div class="pull-right small toast-container">
        <moloch-toast
          class="mr-1"
          :message="msg"
          :type="msgType"
          :done="messageDone">
        </moloch-toast>
      </div>
    </div> <!-- /sub navbar -->

    <!-- loading overlay -->
    <moloch-loading
      v-if="loading && !error">
    </moloch-loading> <!-- /loading overlay -->

    <!-- page error -->
    <moloch-error
      v-if="error"
      :message-html="error"
      class="settings-error">
    </moloch-error> <!-- /page error -->

    <!-- content -->
    <div class="settings-content row ml-1 mr-1"
       v-if="!loading && !error">

      <!-- navigation -->
      <div class="col-md-2 col-sm-3"
        role="tablist"
        aria-orientation="vertical">
        <div class="nav flex-column nav-pills">
          <a class="nav-link cursor-pointer"
            @click="openView('general')"
            :class="{'active':visibleTab === 'general'}">
            <span class="fa fa-fw fa-cog">
            </span>&nbsp;
            General
          </a>
          <a class="nav-link cursor-pointer"
            @click="openView('views')"
            :class="{'active':visibleTab === 'views'}">
            <span class="fa fa-fw fa-eye">
            </span>&nbsp;
            Views
          </a>
          <a class="nav-link cursor-pointer"
            @click="openView('cron')"
            :class="{'active':visibleTab === 'cron'}">
            <span class="fa fa-fw fa-search">
            </span>&nbsp;
            Cron Queries
          </a>
          <a class="nav-link cursor-pointer"
            @click="openView('col')"
            :class="{'active':visibleTab === 'col'}">
            <span class="fa fa-fw fa-columns">
            </span>&nbsp;
            Column Configs
          </a>
          <a class="nav-link cursor-pointer"
            @click="openView('spiview')"
            :class="{'active':visibleTab === 'spiview'}">
            <span class="fa fa-fw fa-eyedropper">
            </span>&nbsp;
            SPI View Configs
          </a>
          <a class="nav-link cursor-pointer"
            @click="openView('theme')"
            :class="{'active':visibleTab === 'theme'}">
            <span class="fa fa-fw fa-paint-brush">
            </span>&nbsp;
            Themes
          </a>
          <a class="nav-link cursor-pointer"
            @click="openView('password')"
            :class="{'active':visibleTab === 'password'}">
            <span class="fa fa-fw fa-lock">
            </span>&nbsp;
            Password
          </a>
        </div>
      </div> <!-- /navigation -->

      <div class="col">

        <!-- general settings -->
        <form class="form-horizontal"
          v-if="visibleTab === 'general'"
          id="general">

          <h3>General</h3>

          <hr>

          <!-- timezone -->
          <div class="form-group row">
            <label class="col-sm-3 col-form-label text-right font-weight-bold">
              Timezone Format
            </label>
            <div class="col-sm-9">
              <div class="btn-group">
                <b-form-group>
                  <b-form-radio-group
                    size="sm"
                    buttons
                    @change="updateTime"
                    v-model="settings.timezone">
                    <b-radio value="local"
                      v-b-tooltip.hover
                      class="btn-radio">
                      Local
                    </b-radio>
                    <b-radio value="gmt"
                      v-b-tooltip.hover
                      class="btn-radio">
                      GMT
                    </b-radio>
                  </b-form-radio-group>
                </b-form-group>
              </div>
              <label class="ml-4 font-weight-bold text-theme-primary">
                {{ date | timezoneDateString(settings.timezone, 'YYYY/MM/DD HH:mm:ss z') }}
              </label>
            </div>
          </div> <!-- /timezone -->

          <!-- session detail format -->
          <div class="form-group row">
            <label class="col-sm-3 col-form-label text-right font-weight-bold">
              Session Detail Format
            </label>
            <div class="col-sm-9">
              <b-form-group>
                <b-form-radio-group
                  size="sm"
                  buttons
                  @change="updateSessionDetailFormat"
                  v-model="settings.detailFormat">
                  <b-radio value="last"
                    v-b-tooltip.hover
                    class="btn-radio">
                    Last Used
                  </b-radio>
                  <b-radio value="natural"
                    v-b-tooltip.hover
                    class="btn-radio">
                    Natural
                  </b-radio>
                  <b-radio value="ascii"
                    v-b-tooltip.hover
                    class="btn-radio">
                    ASCII
                  </b-radio>
                  <b-radio value="utf8"
                    v-b-tooltip.hover
                    class="btn-radio">
                    UTF-8
                  </b-radio>
                  <b-radio value="hex"
                    v-b-tooltip.hover
                    class="btn-radio">
                    Hex
                  </b-radio>
                </b-form-radio-group>
              </b-form-group>
            </div>
          </div> <!-- /session detail format -->

          <!-- number of packets -->
          <div class="form-group row">
            <label class="col-sm-3 col-form-label text-right font-weight-bold">
              Number of Packets
            </label>
            <div class="col-sm-9">
              <b-form-group>
                <b-form-radio-group
                  size="sm"
                  buttons
                  @change="updateNumberOfPackets"
                  v-model="settings.numPackets">
                  <b-radio value="last"
                    v-b-tooltip.hover
                    class="btn-radio">
                    Last Used
                  </b-radio>
                  <b-radio value="50"
                    v-b-tooltip.hover
                    class="btn-radio">
                    50
                  </b-radio>
                  <b-radio value="200"
                    v-b-tooltip.hover
                    class="btn-radio">
                    200
                  </b-radio>
                  <b-radio value="500"
                    v-b-tooltip.hover
                    class="btn-radio">
                    500
                  </b-radio>
                  <b-radio value="1000"
                    v-b-tooltip.hover
                    class="btn-radio">
                    1,000
                  </b-radio>
                  <b-radio value="2000"
                    v-b-tooltip.hover
                    class="btn-radio">
                    2,000
                  </b-radio>
                </b-form-radio-group>
              </b-form-group>
            </div>
          </div> <!-- /number of packets -->

          <!-- show packet timestamp -->
          <div class="form-group row">
            <label class="col-sm-3 col-form-label text-right font-weight-bold">
              Show Packet Timestamps
            </label>
            <div class="col-sm-9">
              <b-form-group>
                <b-form-radio-group
                  size="sm"
                  buttons
                  @change="updateShowPacketTimestamps"
                  v-model="settings.showTimestamps">
                  <b-radio value="last"
                    v-b-tooltip.hover
                    class="btn-radio">
                    Last Used
                  </b-radio>
                  <b-radio value="on"
                    v-b-tooltip.hover
                    class="btn-radio">
                    On
                  </b-radio>
                  <b-radio value="off"
                    v-b-tooltip.hover
                    class="btn-radio">
                    Off
                  </b-radio>
                </b-form-radio-group>
              </b-form-group>
            </div>
          </div> <!-- /show packet timestamp -->

          <!-- issue query on initial page load -->
          <div class="form-group row">
            <label class="col-sm-3 col-form-label text-right font-weight-bold">
              Issue Query on Page Load
            </label>
            <div class="col-sm-9">
              <b-form-group>
                <b-form-radio-group
                  size="sm"
                  buttons
                  @change="updateQueryOnPageLoad"
                  v-model="settings.manualQuery">
                  <b-radio value="false"
                    v-b-tooltip.hover
                    class="btn-radio">
                    Yes
                  </b-radio>
                  <b-radio value="true"
                    v-b-tooltip.hover
                    class="btn-radio">
                    No
                  </b-radio>
                </b-form-radio-group>
              </b-form-group>
            </div>
          </div> <!-- /issue query on initial page load -->

          <!-- session sort -->
          <div class="form-group row">
            <label class="col-sm-3 col-form-label text-right font-weight-bold">
              Sort Sessions By
            </label>
            <div class="col-sm-6">
              <select class="form-control form-control-sm"
                v-model="settings.sortColumn"
                @change="update">
                <option value="last">Last Used</option>
                <option v-for="field in columns"
                  :key="field.dbField"
                  v-if="!field.unsortable"
                  :value="field.dbField">
                  {{ field.friendlyName }}
                </option>
              </select>
            </div>
            <div class="col-sm-3">
              <b-form-group>
                <b-form-radio-group
                  v-if="settings.sortColumn !== 'last'"
                  size="sm"
                  buttons
                  @change="updateSortDirection"
                  v-model="settings.sortDirection">
                  <b-radio value="asc"
                    v-b-tooltip.hover
                    class="btn-radio">
                    ascending
                  </b-radio>
                  <b-radio value="desc"
                    v-b-tooltip.hover
                    class="btn-radio">
                    descending
                  </b-radio>
                </b-form-radio-group>
              </b-form-group>
            </div>
          </div> <!-- /session sort -->

          <!-- default spi graph -->
          <div v-if="fields && settings.spiGraph"
            class="form-group row">
            <label class="col-sm-3 col-form-label text-right font-weight-bold">
              Default SPI Graph
            </label>
            <div class="col-sm-6">
              <moloch-field-typeahead
                :fields="fields"
                query-param="field"
                :initial-value="spiGraphTypeahead"
                @fieldSelected="spiGraphFieldSelected">
              </moloch-field-typeahead>
            </div>
            <div class="col-sm-3">
              <h4 v-if="spiGraphField">
                <label class="badge badge-info cursor-help"
                  v-b-tooltip.hover
                  :title="spiGraphField.help">
                  {{ spiGraphTypeahead || 'unknown field' }}
                </label>
              </h4>
            </div>
          </div> <!-- /default spi graph -->

          <!-- connections src field -->
          <div v-if="fields && settings.connSrcField"
            class="form-group row">
            <label class="col-sm-3 col-form-label text-right font-weight-bold">
              Connections Src
            </label>
            <div class="col-sm-6">
              <moloch-field-typeahead
                :fields="fields"
                query-param="field"
                :initial-value="connSrcFieldTypeahead"
                @fieldSelected="connSrcFieldSelected">
              </moloch-field-typeahead>
            </div>
            <div class="col-sm-3">
              <h4 v-if="connSrcField">
                <label class="badge badge-info cursor-help"
                  v-b-tooltip.hover
                  :title="connSrcField.help">
                  {{ connSrcFieldTypeahead || 'unknown field' }}
                </label>
              </h4>
            </div>
          </div> <!-- /connections src field -->

          <!-- connections dst field -->
          <div v-if="fields && settings.connDstField"
            class="form-group row">
            <label class="col-sm-3 col-form-label text-right font-weight-bold">
              Connections Dst
            </label>
            <div class="col-sm-6">
              <moloch-field-typeahead
                :fields="fieldsPlus"
                query-param="field"
                :initial-value="connDstFieldTypeahead"
                @fieldSelected="connDstFieldSelected">
              </moloch-field-typeahead>
            </div>
            <div class="col-sm-3">
              <h4 v-if="connDstField">
                <label class="badge badge-info cursor-help"
                  v-b-tooltip.hover
                  :title="connDstField.help">
                  {{ connDstFieldTypeahead || 'unknown field' }}
                </label>
              </h4>
            </div>
          </div> <!-- /connections dst field -->

        </form> <!-- / general settings -->

        <!-- view settings -->
        <form v-if="visibleTab === 'views'"
          id="views"
          class="form-horizontal">

          <h3>Views</h3>

          <p>
             Saved views provide an easier method to specify common base queries
             and can be activated in the search bar.
          </p>

          <hr>

          <table class="table table-striped table-sm">
            <thead>
              <tr>
                <th>Name</th>
                <th>Expression</th>
                <th>&nbsp;</th>
              </tr>
            </thead>
            <tbody>
              <!-- views -->
              <tr v-for="(item, key) in views"
                @keyup.enter="updateView(key)"
                @keyup.esc="cancelViewChange(key)"
                :key="key">
                <td>
                  <input type="text"
                    maxlength="20"
                    v-model="item.name"
                    @input="viewChanged(key)"
                    class="form-control form-control-sm"
                  />
                </td>
                <td>
                  <input type="text"
                    v-model="item.expression"
                    @input="viewChanged(key)"
                    class="form-control form-control-sm"
                  />
                </td>
                <td>
                  <div class="btn-group btn-group-sm pull-right"
                    v-if="item.changed">
                    <button type="button"
                      v-b-tooltip.hover
                      @click="updateView(key)"
                      title="Save changes to this view"
                      class="btn btn-theme-tertiary">
                      <span class="fa fa-save">
                      </span>
                    </button>
                    <button type="button"
                      v-b-tooltip.hover
                      class="btn btn-warning"
                      @click="cancelViewChange(key)"
                      title="Undo changes to this view">
                      <span class="fa fa-ban">
                      </span>
                    </button>
                  </div>
                  <button v-else
                    type="button"
                    class="btn btn-sm btn-danger pull-right"
                    @click="deleteView(key)">
                    <span class="fa fa-trash-o">
                    </span>&nbsp;
                    Delete
                  </button>
                </td>
              </tr> <!-- /views -->
              <!-- view list error -->
              <tr v-if="viewListError">
                <td colspan="3">
                  <p class="text-danger mb-0">
                    <span class="fa fa-exclamation-triangle">
                    </span>&nbsp;
                    {{ viewListError }}
                  </p>
                </td>
              </tr> <!-- /view list error -->
              <!-- new view form -->
              <tr @keyup.enter="createView">
                <td>
                  <input type="text"
                    maxlength="20"
                    v-model="newViewName"
                    class="form-control form-control-sm"
                    placeholder="Enter a new view name (20 chars or less)"
                  />
                </td>
                <td>
                  <input type="text"
                    v-model="newViewExpression"
                    class="form-control form-control-sm"
                    placeholder="Enter a new view expression"
                  />
                </td>
                <td>
                  <button class="btn btn-theme-tertiary btn-sm pull-right"
                    type="button"
                    @click="createView()">
                    <span class="fa fa-plus-circle">
                    </span>&nbsp;
                    Create
                  </button>
                </td>
              </tr> <!-- /new view form -->
              <!-- view form error -->
              <tr v-if="viewFormError">
                <td colspan="3">
                  <p class="text-danger mb-0">
                    <span class="fa fa-exclamation-triangle">
                    </span>&nbsp;
                    {{ viewFormError }}
                  </p>
                </td>
              </tr> <!-- /view form error -->
            </tbody>
          </table>

        </form> <!-- /view settings -->

      </div>

    </div> <!-- /content -->

  </div> <!-- /settings content -->

</template>

<script>
import UserService from '../users/UserService';
import ConfigService from '../utils/ConfigService';
import FieldService from '../search/FieldService';
import SessionsService from '../sessions/SessionsService';
import customCols from '../sessions/customCols.json';
import MolochToast from '../utils/Toast';
import MolochError from '../utils/Error';
import MolochLoading from '../utils/Loading';
import MolochFieldTypeahead from '../utils/FieldTypeahead';

let clockInterval;

const defaultSpiviewConfig = { fields: ['dstIp', 'protocol', 'srcIp'] };
const defaultColConfig = {
  order: [['firstPacket', 'asc']],
  columns: ['firstPacket', 'lastPacket', 'src', 'srcPort', 'dst', 'dstPort', 'totPackets', 'dbby', 'node', 'info']
};

export default {
  name: 'Settings',
  components: { MolochError, MolochLoading, MolochToast, MolochFieldTypeahead },
  data: function () {
    return {
      // page vars
      error: '',
      loading: true,
      msg: '',
      msgType: undefined,
      displayName: undefined,
      visibleTab: 'general', // default tab
      settings: {},
      fields: undefined,
      fieldsMap: undefined,
      fieldsPlus: undefined,
      columns: [],
      // general settings vars
      date: undefined,
      spiGraphField: undefined,
      spiGraphTypeahead: undefined,
      connSrcField: undefined,
      connSrcFieldTypeahead: undefined,
      connDstField: undefined,
      connDstFieldTypeahead: undefined,
      // view settings vars
      views: {},
      viewListError: '',
      viewFormError: '',
      newViewName: '',
      newViewExpression: '',

      defaultColConfig: defaultColConfig,
      defaultSpiviewConfig: defaultSpiviewConfig,
      newCronQueryProcess: '0',
      newCronQueryAction: 'tag',
      themeDisplays: [
        { name: 'Purp-purp', class: 'default-theme' },
        { name: 'Blue', class: 'blue-theme' },
        { name: 'Green', class: 'green-theme' },
        { name: 'Cotton Candy', class: 'cotton-candy-theme' },
        { name: 'Green on Black', class: 'dark-2-theme' },
        { name: 'Dark Blue', class: 'dark-3-theme' }
      ],
      cronQueries: undefined,
      cronQueryListError: '',
      colConfigs: undefined,
      colConfigError: '',
      spiviewConfigs: undefined,
      spiviewConfigError: '',
      molochClusters: {}
    };
  },
  created: function () {
    // does the url specify a tab in hash
    let tab = window.location.hash;
    if (tab) { // if there is a tab specified and it's a valid tab
      tab = tab.replace(/^#/, '');
      if (tab === 'general' || tab === 'views' || tab === 'cron' ||
        tab === 'col' || tab === 'theme' || tab === 'password' ||
        tab === 'spiview') {
        this.visibleTab = tab;
      }
    }

    this.getThemeColors();

    UserService.getCurrent()
      .then((response) => {
        this.displayName = response.userId;
        // only admins can edit other users' settings
        if (response.createEnabled && this.$route.query.userId) {
          if (response.userId === this.$route.query.userId) {
            // admin editing their own user so the routeParam is unnecessary
            this.$router.push({
              hash: this.$route.hash,
              query: {
                ...this.$route.query,
                userId: undefined
              }
            });
          } else { // admin editing another user
            this.userId = this.$route.query.userId;
            this.displayName = this.$route.query.userId;
          }
        } else { // normal user has no permission, so remove the routeParam
          // (even if it's their own userId because it's unnecessary)
          this.$router.push({
            hash: this.$route.hash,
            query: {
              ...this.$route.query,
              userId: undefined
            }
          });
        }

        // always get the user's settings because current user is cached
        // so response.settings might be stale
        this.getSettings();

        // get all the other things!
        this.getViews();
        this.getCronQueries();
        this.getColConfigs();
        this.getSpiviewConfigs();
      })
      .catch((error) => {
        this.error = error.text;
        this.loading = false;
      });

    ConfigService.getMolochClusters()
      .then((response) => {
        this.molochClusters = response;
      });

    // get fields from field service then get sessionsNew state
    FieldService.get(true)
      .then((response) => {
        this.fields = response;
        this.fieldsPlus = JSON.parse(JSON.stringify(response));
        this.fieldsPlus.push({
          dbField: 'ip.dst:port',
          exp: 'ip.dst:port',
          help: 'Destination IP:Destination Port',
          group: 'general',
          friendlyName: 'Dst IP:Dst Port'
        });

        // add custom columns to the fields array
        for (let key in customCols) {
          if (customCols.hasOwnProperty(key)) {
            this.fields.push(customCols[key]);
          }
        }

        // build fields map for quick lookup by dbField
        this.fieldsMap = {};
        for (let i = 0, len = this.fields.length; i < len; ++i) {
          let field = this.fields[i];
          this.fieldsMap[field.dbField] = field;
        }

        SessionsService.getState('sessionsNew')
          .then((response) => {
            this.setupColumns(response.data.visibleHeaders);
            // if the sort column setting does not match any of the visible
            // headers, set the sort column setting to last
            if (response.data.visibleHeaders.indexOf(this.settings.sortColumn) === -1) {
              this.settings.sortColumn = 'last';
            }
          })
          .catch(() => {
            this.setupColumns(['firstPacket', 'lastPacket', 'src', 'srcPort', 'dst', 'dstPort', 'totPackets', 'dbby', 'node', 'info']);
          });
      });
  },
  methods: {
    /* exposed page functions ---------------------------------------------- */
    /* opens a specific settings tab */
    openView: function (tabName) {
      this.visibleTab = tabName;
      this.$router.push({
        hash: tabName
      });
    },
    /* remove the message when user is done with it or duration ends */
    messageDone: function () {
      this.msg = '';
      this.msgType = undefined;
    },
    /* GENERAL ------------------------------- */
    /**
     * saves the user's settings and displays a message
     * @param updateTheme whether to update the UI theme
     */
    update: function (updateTheme) {
      UserService.saveSettings(this.settings, this.userId)
        .then((response) => {
          // display success message to user
          this.msg = response.text;
          this.msgType = 'success';

          if (updateTheme) {
            let now = Date.now();
            if ($('link[href^="user.css"]').length) {
              $('link[href^="user.css"]').remove();
            }
            $('head').append(`<link rel="stylesheet"
                              href="user.css?v${now}"
                              type="text/css" />`);
          }
        })
        .catch((error) => {
          // display error message to user
          this.msg = error.text;
          this.msgType = 'danger';
        });
    },
    /* updates the displayed date for the timzeone setting
     * triggered by the user changing the timezone setting */
    updateTime: function (newTimezone) {
      this.settings.timezone = newTimezone;
      this.tick();
      this.update();
    },
    updateSessionDetailFormat: function (newDetailFormat) {
      this.settings.detailFormat = newDetailFormat;
      this.update();
    },
    updateNumberOfPackets: function (newNumPackets) {
      this.settings.numPackets = newNumPackets;
      this.update();
    },
    updateShowPacketTimestamps: function (newShowTimestamps) {
      this.settings.showTimestamps = newShowTimestamps;
      this.update();
    },
    updateQueryOnPageLoad: function (newManualQuery) {
      this.settings.manualQuery = newManualQuery;
      this.update();
    },
    updateSortDirection: function (newSortDirection) {
      this.settings.sortDirection = newSortDirection;
      this.update();
    },
    spiGraphFieldSelected: function (field) {
      this.spiGraphTypeahead = field.friendlyName;
      this.settings.spiGraph = field.dbField;
      this.spiGraphField = field;
      this.update();
    },
    connSrcFieldSelected: function (field) {
      this.connSrcFieldTypeahead = field.friendlyName;
      this.settings.connSrcField = field.dbField;
      this.connSrcField = field;
      this.update();
    },
    connDstFieldSelected: function (field) {
      this.connDstFieldTypeahead = field.friendlyName;
      this.settings.connDstField = field.dbField;
      this.connDstField = field;
      this.update();
    },
    /* starts the clock for the timezone setting */
    startClock: function () {
      this.tick();
      clockInterval = setInterval(() => {
        this.tick();
      }, 1000);
    },
    /* updates the date and format for the timezone setting */
    tick: function () {
      this.date = Math.floor(new Date() / 1000);
      if (this.settings.timezone === 'gmt') {
        this.dateFormat = 'yyyy/MM/dd HH:mm:ss\'Z\'';
      } else {
        this.dateFormat = 'yyyy/MM/dd HH:mm:ss';
      }
    },
    /**
     * Displays the field.exp instead of field.dbField in the
     * field typeahead inputs
     * @param {string} value The dbField of the field
     */
    formatField: function (value) {
      for (let i = 0, len = this.fields.length; i < len; i++) {
        if (value === this.fields[i].dbField) {
          return this.fields[i].friendlyName;
        }
      }
    },
    /* VIEWS ------------------------------------------- */
    /* creates a view given the view name and expression */
    createView: function () {
      if (!this.newViewName || this.newViewName === '') {
        this.viewFormError = 'No view name specified.';
        return;
      }

      if (!this.newViewExpression || this.newViewExpression === '') {
        this.viewFormError = 'No view expression specified.';
        return;
      }

      let data = {
        viewName: this.newViewName,
        expression: this.newViewExpression
      };

      UserService.createView(data, this.userId)
        .then((response) => {
          // add the view to the view list
          this.views[data.viewName] = {
            expression: data.expression,
            name: data.viewName
          };
          this.viewFormError = false;
          // clear the inputs
          this.newViewName = null;
          this.newViewExpression = null;
          // display success message to user
          this.msg = response.text;
          this.msgType = 'success';
        })
        .catch((error) => {
          // display error message to user
          this.msg = error.text;
          this.msgType = 'danger';
        });
    },
    /**
     * Deletes a view given its name
     * @param {string} name The name of the view to delete
     */
    deleteView: function (name) {
      UserService.deleteView(name, this.userId)
        .then((response) => {
          // remove the view from the view list
          this.views[name] = null;
          delete this.views[name];
          // display success message to user
          this.msg = response.text;
          this.msgType = 'success';
        })
        .catch((error) => {
          // display error message to user
          this.msg = error.text;
          this.msgType = 'danger';
        });
    },
    /**
     * Sets a view as having been changed
     * @param {string} key The unique id of the changed view
     */
    viewChanged: function (key) {
      this.views[key].changed = true;
    },
    /**
     * Cancels a view change by retrieving the view
     * @param {string} key The unique id of the view
     */
    cancelViewChange: function (key) {
      UserService.getViews(this.userId)
        .then((response) => {
          this.views[key] = response[key];
        })
        .catch((error) => {
          this.viewListError = error.text;
        });
    },
    /**
     * Updates a view
     * @param {string} key The unique id of the view to update
     */
    updateView: function (key) {
      let data = this.views[key];

      if (!data) {
        this.msg = 'Could not find corresponding view';
        this.msgType = 'danger';
        return;
      }

      if (!data.changed) {
        this.msg = 'This view has not changed';
        this.msgType = 'warning';
        return;
      }

      data.key = key;

      UserService.updateView(data, this.userId)
        .then((response) => {
          // update view list
          this.views = response.views;
          // display success message to user
          this.msg = response.text;
          this.msgType = 'success';
          // set the view as unchanged
          data.changed = false;
        })
        .catch((error) => {
          // display error message to user
          this.msg = error.text;
          this.msgType = 'danger';
        });
    },
    /* helper functions ---------------------------------------------------- */
    /* retrievs the theme colors from the document body's property values */
    getThemeColors: function () {
      let styles = window.getComputedStyle(document.body);

      this.background = styles.getPropertyValue('--color-background').trim() || '#FFFFFF';
      this.foreground = styles.getPropertyValue('--color-foreground').trim() || '#333333';
      this.foregroundAccent = styles.getPropertyValue('--color-foreground-accent').trim();

      this.primary = styles.getPropertyValue('--color-primary').trim();
      this.primaryLightest = styles.getPropertyValue('--color-primary-lightest').trim();

      this.secondary = styles.getPropertyValue('--color-secondary').trim();
      this.secondaryLightest = styles.getPropertyValue('--color-secondary-lightest').trim();

      this.tertiary = styles.getPropertyValue('--color-tertiary').trim();
      this.tertiaryLightest = styles.getPropertyValue('--color-tertiary-lightest').trim();

      this.quaternary = styles.getPropertyValue('--color-quaternary').trim();
      this.quaternaryLightest = styles.getPropertyValue('--color-quaternary-lightest').trim();

      this.water = styles.getPropertyValue('--color-water').trim();
      this.land = styles.getPropertyValue('--color-land').trim() || this.primary;

      this.src = styles.getPropertyValue('--color-src').trim() || '#CA0404';
      this.dst = styles.getPropertyValue('--color-dst').trim() || '#0000FF';

      this.setThemeString();
    },
    setThemeString: function () {
      this.themeString = `${this.background},${this.foreground},${this.foregroundAccent},${this.primary},${this.primaryLightest},${this.secondary},${this.secondaryLightest},${this.tertiary},${this.tertiaryLightest},${this.quaternary},${this.quaternaryLightest},${this.water},${this.land},${this.src},${this.dst}`;
    },
    /* retrieves the specified user's settings */
    getSettings: function () {
      UserService.getSettings(this.userId)
        .then((response) => {
          // set defaults so that radio buttons show the default value
          if (!response.timezone) { response.timezone = 'local'; }
          if (!response.detailFormat) { response.detailFormat = 'last'; }
          if (!response.numPackets) { response.numPackets = 'last'; }
          if (!response.showTimestamps) { response.showTimestamps = 'last'; }
          if (!response.manualQuery) { response.manualQuery = false; }

          // dbField is saved in settings, but show the field's friendlyName
          for (let i = 0, len = this.fieldsPlus.length; i < len; i++) {
            let field = this.fieldsPlus[i];
            if (response.spiGraph === field.dbField) {
              this.spiGraphField = field;
              this.spiGraphTypeahead = field.friendlyName;
            }
            if (response.connSrcField === field.dbField) {
              this.connSrcField = field;
              this.connSrcFieldTypeahead = field.friendlyName;
            }
            if (response.connDstField === field.dbField) {
              this.connDstField = field;
              this.connDstFieldTypeahead = field.friendlyName;
            }
          }

          this.settings = response;
          this.loading = false;

          this.setTheme();
          this.startClock();
        })
        .catch((error) => {
          this.loading = false;
          if (error.text === 'User not found') {
            this.error = `<div class="text-center">
                            ${error.text}
                            <small><a href="settings">View your own settings?</a></small>
                          </div>`;
          } else {
            this.error = error.text;
          }
          this.displayName = '';
        });
    },
    /* retrieves the specified user's views */
    getViews: function () {
      UserService.getViews(this.userId)
        .then((response) => {
          this.views = response;
        })
        .catch((error) => {
          this.viewListError = error.text;
        });
    },
    /* retrieves the specified user's cron queries */
    getCronQueries: function () {
      UserService.getCronQueries(this.userId)
        .then((response) => {
          this.cronQueries = response;
        })
        .catch((error) => {
          this.cronQueryListError = error.text;
        });
    },
    /* retrieves the specified user's custom column configurations */
    getColConfigs: function () {
      UserService.getColumnConfigs(this.userId)
        .then((response) => {
          this.colConfigs = response;
        })
        .catch((error) => {
          this.colConfigError = error.text;
        });
    },
    /* retrieves the specified user's custom spiview fields configurations.
     * dissects the visible spiview fields for view consumption */
    getSpiviewConfigs: function () {
      UserService.getSpiviewFields(this.userId)
        .then((response) => {
          this.spiviewConfigs = response;

          for (let x = 0, xlen = this.spiviewConfigs.length; x < xlen; ++x) {
            let config = this.spiviewConfigs[x];
            let spiParamsArray = config.fields.split(',');

            // get each field from the spi query parameter and issue
            // a query for one field at a time
            for (let i = 0, len = spiParamsArray.length; i < len; ++i) {
              let param = spiParamsArray[i];
              let split = param.split(':');
              let fieldID = split[0];
              let count = split[1];

              let field;

              for (let key in this.fields) {
                if (this.fields[key].dbField === fieldID) {
                  field = this.fields[key];
                  break;
                }
              }

              if (field) {
                if (!config.fieldObjs) { config.fieldObjs = []; }

                field.count = count;
                config.fieldObjs.push(field);
              }
            }
          }
        })
        .catch((error) => {
          this.spiviewConfigError = error.text;
        });
    },
    /**
     * Setup this.columns with a list of field objects
     * @param {array} colIdArray The array of column ids
     */
    setupColumns: function (colIdArray) {
      this.columns = [];
      for (let i = 0, len = colIdArray.length; i < len; ++i) {
        this.columns.push(this.getField(colIdArray[i]));
      }
    },
    /**
     * Gets the field that corresponds to a field's dbField value
     * @param {string} dbField The fields dbField value
     * @returns {object} field The field that corresponds to the entered dbField
     */
    getField: function (dbField) {
      return this.fieldsMap[dbField];
    },
    /* THEMES -------------------------------------------------------------- */
    setTheme: function () {
      // default to default theme if the user has not set a theme
      if (!this.settings.theme) { this.settings.theme = 'default-theme'; }
      if (this.settings.theme.startsWith('custom')) {
        this.settings.theme = 'custom-theme';
        this.creatingCustom = true;
      }
    }
  },
  beforeDestroy: function () {
    if (clockInterval) { clearInterval(clockInterval); }
  }
};
</script>

<style>
/* fix dropdown location */
.settings-page .dropdown-menu.field-typeahead {
  margin-top: -15px;
  left: 15px;
}
</style>

<style scoped>
/* settings page, navbar, and content styles - */
.settings-page {
  margin-top: 36px;
}
.settings-content {
  margin-top: 90px;
}

.settings-page .sub-navbar {
  z-index: 2;
}

/* fixed tab buttons */
.settings-page div.nav-pills {
  position: fixed;
}

/* make sure the form is taller than the nav pills */
.settings-page form {
  min-height: 280px;
}

.settings-page .settings-error {
  margin-top: 6rem;
  margin-bottom: 1rem;
}
</style>
