clockInterval<template>

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
      :message="error"
      class="mt-5 mb-5">
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

      <div class="col mt-5">
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

let clockInterval;

const defaultSpiviewConfig = { fields: ['dstIp', 'protocol', 'srcIp'] };
const defaultColConfig = {
  order: [['firstPacket', 'asc']],
  columns: ['firstPacket', 'lastPacket', 'src', 'srcPort', 'dst', 'dstPort', 'totPackets', 'dbby', 'node', 'info']
};

export default {
  name: 'Settings',
  components: { MolochError, MolochLoading, MolochToast },
  data: function () {
    return {
      error: '',
      loading: true,
      msg: '',
      msgType: undefined,
      displayName: undefined,
      visibleTab: 'general', // default tab
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
      views: undefined,
      viewListError: '',
      cronQueries: undefined,
      cronQueryListError: '',
      colConfigs: undefined,
      colConfigError: '',
      spiviewConfigs: undefined,
      spiviewConfigError: '',
      molochClusters: {},
      fields: undefined,
      fieldsMap: undefined,
      fieldsPlus: undefined,
      columns: []
    };
  },
  created: function () {
    // does the url specify a tab in hash
    let tab = window.location.hash;
    if (tab) { // if there is a tab specified and it's a valid tab
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
        if (response.createEnabled && this.$routeParams.userId) {
          if (response.userId === this.$routeParams.userId) {
            // admin editing their own user so the routeParam is unnecessary
            // TODO test
            this.$router.push({
              query: {
                ...this.$route.query,
                userId: undefined
              }
            });
          } else { // admin editing another user
            // TODO test
            this.userId = this.$route.query.userId;
            this.displayName = this.$route.query.userId;
          }
        } else { // normal user has no permission, so remove the routeParam
          // (even if it's their own userId because it's unnecessary)
          // TODO test
          this.$router.push({
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
        this.fieldsPlus = response;
        this.fieldsPlus.push({
          dbField: 'ip.dst:port',
          exp: 'ip.dst:port',
          help: 'Destination IP:Destination Port'
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
            if (response.data.visibleHeaders.indexOf(this.settings.sortColumn === -1)) {
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
      // TODO test
      window.location.hash = tabName;
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
      // TODO test
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
    updateTime: function () {
      this.tick();
      this.update();
    },
    /**
     * Displays the field.exp instead of field.dbField in the
     * field typeahead inputs
     * @param {string} value The dbField of the field
     */
    formatField: function (value) {
      // TODO need this?
      for (let i = 0, len = this.fields.length; i < len; i++) {
        if (value === this.fields[i].dbField) {
          return this.fields[i].exp;
        }
      }
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
    /* starts the clock for the timezone setting */
    startClock: function () {
      this.tick();
      clockInterval = setInterval(() => {
        this.tick();
      }, 1000);
    },
    /* updates the date and format for the timezone setting */
    tick: function () {
      // TODO
      this.date = new Date();
      if (this.settings.timezone === 'gmt') {
        this.dateFormat = 'yyyy/MM/dd HH:mm:ss\'Z\'';
      } else {
        this.dateFormat = 'yyyy/MM/dd HH:mm:ss';
      }
    },
    /* retrieves the specified user's settings */
    getSettings: function () {
      UserService.getSettings(this.userId)
        .then((response) => {
          this.settings = response;
          this.loading = false;

          this.setTheme();
          this.startClock();
        })
        .catch((error) => {
          this.loading = false;
          this.error = error.data.text;
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
      for (let i = 0, len = this.fields.length; i < len; i++) {
        if (dbField === this.fields[i].dbField) {
          return this.fields[i];
        }
      }
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

<style scoped>
/* settings page, navbar, and content styles - */
.settings-page {
  margin-top: 36px;
}
.settings-content {
  margin-top: 90px;
}
</style>
