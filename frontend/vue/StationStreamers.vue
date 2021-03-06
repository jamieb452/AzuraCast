<template>
    <div>
        <b-card no-body>
            <b-card-header header-bg-variant="primary-dark">
                <b-row class="align-items-center">
                    <b-col md="6">
                        <h2 class="card-title" v-translate>Streamer/DJ Accounts</h2>
                    </b-col>
                    <b-col md="6" class="text-right text-muted">
                        <translate :translate-params="{ tz: stationTimeZone }">This station's time zone is currently %{tz}.</translate>
                    </b-col>
                </b-row>
            </b-card-header>

            <b-tabs pills card lazy>
                <b-tab :title="langAccountListTab" no-body>
                    <b-card-body body-class="card-padding-sm">
                        <b-button variant="outline-primary" @click.prevent="doCreate">
                            <i class="material-icons" aria-hidden="true">add</i>
                            <translate>Add Streamer</translate>
                        </b-button>
                    </b-card-body>

                    <div class="table-responsive table-responsive-lg">
                        <data-table ref="datatable" id="station_streamers" :show-toolbar="false" :fields="fields"
                                    :api-url="listUrl">
                            <template v-slot:cell(streamer_username)="row">
                                <code>{{ row.item.streamer_username }}</code>
                                <div>
                                    <span class="badge badge-danger" v-if="!row.item.is_active">
                                        <translate>Disabled</translate>
                                    </span>
                                </div>
                            </template>
                            <template v-slot:cell(actions)="row">
                                <b-button-group size="sm">
                                    <b-button size="sm" variant="primary" @click.prevent="doEdit(row.item.links.self)">
                                        <translate>Edit</translate>
                                    </b-button>
                                    <b-button size="sm" variant="default" @click.prevent="doShowBroadcasts(row.item.links.broadcasts)">
                                        <translate>Broadcasts</translate>
                                    </b-button>
                                    <b-button size="sm" variant="danger" @click.prevent="doDelete(row.item.links.self)">
                                        <translate>Delete</translate>
                                    </b-button>
                                </b-button-group>
                            </template>
                        </data-table>
                    </div>
                </b-tab>
                <b-tab :title="langScheduleViewTab" no-body>
                    <schedule ref="schedule" :schedule-url="scheduleUrl" :station-time-zone="stationTimeZone"
                              :locale="locale" @click="doCalendarClick"></schedule>
                </b-tab>
            </b-tabs>
        </b-card>

        <edit-modal ref="editModal" :create-url="listUrl" :station-time-zone="stationTimeZone" @relist="relist"></edit-modal>
        <broadcasts-modal ref="broadcastsModal"></broadcasts-modal>
    </div>
</template>

<script>
    import DataTable from './components/DataTable';
    import axios from 'axios';
    import EditModal from './station_streamers/StreamerEditModal';
    import BroadcastsModal from './station_streamers/StreamerBroadcastsModal';
    import Schedule from './components/ScheduleView';

    export default {
        name: 'StationStreamers',
        components: { EditModal, BroadcastsModal, DataTable, Schedule },
        props: {
            listUrl: String,
            scheduleUrl: String,
            filesUrl: String,
            locale: String,
            stationTimeZone: String
        },
        data () {
            return {
                fields: [
                    { key: 'actions', label: this.$gettext('Actions'), sortable: false },
                    { key: 'streamer_username', label: this.$gettext('Username'), sortable: false },
                    { key: 'display_name', label: this.$gettext('Display Name'), sortable: false },
                    { key: 'comments', label: this.$gettext('Notes'), sortable: false }
                ]
            };
        },
        computed: {
            langAccountListTab () {
                return this.$gettext('Account List');
            },
            langScheduleViewTab () {
                return this.$gettext('Schedule View');
            }
        },
        mounted () {
            moment.relativeTimeThreshold('ss', 1);
            moment.relativeTimeRounding(function (value) {
                return Math.round(value * 10) / 10;
            });
        },
        methods: {
            relist () {
                this.$refs.datatable.refresh();
            },
            doCreate () {
                this.$refs.editModal.create();
            },
            doCalendarClick (event) {
                this.doEdit(event.extendedProps.edit_url);
            },
            doEdit (url) {
                this.$refs.editModal.edit(url);
            },
            doShowBroadcasts (url) {
                this.$refs.broadcastsModal.open(url);
            },
            doDelete (url) {
                let buttonText = this.$gettext('Delete');
                let buttonConfirmText = this.$gettext('Delete streamer?');

                swal({
                    title: buttonConfirmText,
                    buttons: [true, buttonText],
                    dangerMode: true
                }).then((value) => {
                    if (value) {
                        axios.delete(url).then((resp) => {
                            notify('<b>' + resp.data.message + '</b>', 'success');

                            this.relist();
                        }).catch((err) => {
                            console.error(err);
                            if (err.response.message) {
                                notify('<b>' + err.response.message + '</b>', 'danger');
                            }
                        });
                    }
                });
            }
        }
    };
</script>