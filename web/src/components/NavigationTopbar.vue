<template>
    <header class="app-header">
        <nav class="navbar navbar-expand-lg navbar-dark">
        <!-- Navbar content -->

            <div class="container-fluid">
                <a class="navbar-brand" href="https://www2.gov.bc.ca" style="max-width:200px">
                    <img
                        class="img-fluid d-none d-md-block"
                        src="../../public/images/bcid-logo-rev-en.svg"
                        width="177"
                        height="44"
                        alt="B.C. Government Logo"
                    />

                    <img
                        class="img-fluid d-md-none"
                        src="../../public/images/bcid-symbol-rev.svg"
                        width="63"
                        height="44"
                        alt="B.C. Government Logo"
                    />
                </a>
                <div class="navbar-brand navbar-text">
                    Apply for a Family Law Act Protection Order
                    <span class="navbar-tag">BETA</span>
                </div>

                <div class="navbar-extra">
                    <div id="app-profile">
                        <div v-if="userName" style="padding-right: rem">
                            <b-dropdown
                                id="profileDropdown"
                                text="Profile"
                                variant="primary btn-transparent"
                                menu-class="w-10"
                                style="margin-right: 1rem"
                            >
                                <template #button-content style="background-color: #003366">
                                    <span class="fa fa-user"></span> {{ userName }}
                                </template>
                                <b-dropdown-item @click="logout(false)">Logout</b-dropdown-item>
                            </b-dropdown>
                        </div>
                    </div>
                </div>
                <div id="app-exit" class="app-exit">
                    <a
                        @click="logout(true)"
                        target="_blank"
                        id="quick-exit"
                        rel="external"
                        class="btn btn-primary text-warning"
                        ><span class="fa fa-sign-out"/> Quick Exit</a
                    >
                </div>

                <button
                class="navbar-toggler"
                type="button"
                data-toggle="collapse"
                data-target="#navbarNavAltMarkup"
                aria-controls="navbarNavAltMarkup"
                aria-expanded="false"
                aria-label="Toggle navigation"
                >
                    <span class="navbar-toggler-icon"></span>
                </button>
            </div>
        </nav>
    </header>
</template>

<script lang="ts">
import { Component, Vue} from 'vue-property-decorator';
import { SessionManager } from "@/components/utils/utils";
import moment from "moment-timezone";

@Component
export default class NavigationTopbar extends Vue {
    
    error = "";

    get userName() {
      return this.$store.state.Application.userName;
    }
  

    public logout(isQuickExit) {

        const emptyApplicationRoutes = ["/", "/status", "/serviceLocator"];

        if (emptyApplicationRoutes.indexOf(this.$route.fullPath) == -1) {
            const lastUpdated = moment().format();
            this.$store.commit("Application/setLastUpdated", lastUpdated);
            const application = this.$store.state.Application;
            const applicationId = application.id;

            const header = {
                responseType: "json",
                headers: {
                "Content-Type": "application/json",
                }
            }

            this.$http.put("/app/" + applicationId + "/", application, header)
            .then(res => {
                //console.log(res.data);
                this.error = "";
            }, err => {
                console.error(err);
                this.error = err;
            });
        }
        Vue.nextTick().then(() => {
            if (isQuickExit){
                window.open('http://www.google.ca');
                SessionManager.logoutAndRedirect(this.$store);
            }else 
                SessionManager.logout(this.$store);
        });
    }

};
</script>

<style>
.btn-transparent {
    background-color: transparent !important;
    border-color: #ccc !important;
}
</style>

<style scoped lang="scss">
@import "../styles/common";

.app-exit + .navbar {
    padding-right: 170px;
}

.navbar-brand:not(.logo) {
    flex: 1 1 auto;
}

.navbar-extra {
    display: inline-block;
    flex: 1 1 auto;
    text-align: right;
}
.navbar .navbar-extra {
    display: inline-block;
    flex: 1 1 auto;
    text-align: right;
}

.app-exit,
.navbar {
    .btn-primary {
        border-color: #ccc;
    }
}



#app-exit {
    padding: 8px 15px;
    position: absolute;
    position: fixed;
    right: 0;
    top: 0;
    z-index: 100;
    .btn {
        border-radius: 10rem;
        font-size: 110%;
        padding: 0.5em 1em;
    }
}

#app-profile {
    color: $gov-white;
}
</style>
