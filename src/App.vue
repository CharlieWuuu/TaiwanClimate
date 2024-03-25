<template>
    <header>
        <p>看見臺灣的氣候</p>
    </header>
    <nav>
        <router-link to="/">首頁</router-link>
        <router-link to="/About">關於</router-link>
    </nav>
    <div class="page">
        <router-view />
    </div>
</template>

<style lang="scss">
body {
    margin: 0;
    padding: 0;

    #app {
        font-family: Avenir, Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;

        header {
            background-color: #fdffc1;
            padding: 4px 0;
            p {
                font-size: 18px;
                font-weight: 500;
                margin: 0;
                padding: 0;
            }
        }

        nav {
            padding: 30px;

            a {
                font-weight: bold;
                color: #2c3e50;

                &.router-link-exact-active {
                    color: #42b983;
                }
            }
        }
    }
}
</style>
