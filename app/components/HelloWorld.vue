<template>
    <Page class="page" @loaded="onPageLoaded">
        <ActionBar title="Home" flat="true" :class="{ completed: activeTabIndex === 1}" />
        <TabView :selectedIndex="activeTabIndex" @selectedIndexChange="onTabChange"
            selectedTabTextColor="#42B883" androidTabsPosition="bottom">
            <TabViewItem title="Todo" textTransform="uppercase" :iconSource="todoIcon">
                <GridLayout rows="auto, auto, *">
                    <Label row="0" textWrap="true" text="My tasks" class="header"></Label>
                    <TextField row="1" v-model="task" ref="textInput" hint="Next thing todo..."
                        returnKeyType="done" @returnPress="onReturnPress" />

                    <ListView row="2" class="list-group" for="todo in todos" @itemLoading="onItemLoading">
                        <v-template>
                            <GridLayout columns="auto, *" class="list-entry">
                                <Label col="0" text="" @tap="onTodoCircleTap(todo)"
                                    class="circle" />
                                <Label col="1" :text="todo.name" textWrap="true"
                                    @tap="onTodoItemTap(todo)" />
                            </GridLayout>
                        </v-template>
                    </ListView>
                </GridLayout>
            </TabViewItem>
            <TabViewItem title="Completed" textTransform="uppercase" :icon="completedIcon">
                <GridLayout rows="auto, *">
                    <Label row="0" textWrap="true" text="Done Tasks"
                        class="header completed" />
                    <ListView row="1" for="todo in completedTasks" class="list-group" @itemLoading="onItemLoading">
                        <v-template>
                            <GridLayout columns="auto, *" class="list-entry list-entry-completed">
                                <Label col="0" text="✓" @tap="onCompletedCircleTap(todo)"
                                    class="circle" />
                                <Label col="1" :text="todo.name" @tap="onCompletedItemTap(todo)"
                                    textWrap="true" />
                            </GridLayout>
                        </v-template>
                    </ListView>
                </GridLayout>
            </TabViewItem>
        </TabView>
    </Page>
</template>

<script>
    const platform = require("tns-core-modules/platform");
    const frame = require("tns-core-modules/ui/frame");
    export default {
        methods: {
            onPageLoaded() {
                if (platform.isIOS) {
                    const navBar = frame.topmost().ios.controller.navigationBar;
                    navBar.barStyle = UIBarStyle.UIBarStyleBlack;
                    IQKeyboardManager.sharedManager().enableAutoToolbar =
                        false;
                }
            },
            onItemLoading(args) {
                if (args.ios) {
                    args.ios.selectionStyle = UITableViewCellSelectionStyle.None;
                }
            },
            onTodoItemTap: function(todo) {
                const index = this.todos.indexOf(todo);
                action("What do you want to do with this task?", "Cancel",
                    [
                        "Mark Completed",
                        "Delete Forever"
                    ]).then(result => {
                    console.log(result);
                    switch (result) {
                        case "Mark Completed":
                            this.completedTasks.unshift(todo);
                            this.todos.splice(index, 1);
                            this.activeTabIndex = 1;
                            break;
                        case "Delete Forever":
                            this.todos.splice(index, 1);
                            break;
                        case "Cancel" || undefined:
                            break;
                    }
                });
            },

            onTodoCircleTap: function(todo) {
                const index = this.todos.indexOf(todo);
                this.completedTasks.unshift(todo);
                this.todos.splice(index, 1);
                this.activeTabIndex = 1;
            },

            onCompletedItemTap: function(todo) {
                const index = this.completedTasks.indexOf(todo);
                action("What do you want to do with this task?", "Cancel",
                    [
                        "Mark Completed",
                        "Delete Forever"
                    ]).then(result => {
                    console.log(result);
                    switch (result) {
                        case "Mark Completed":
                            this.todos.unshift(todo);
                            this.completedTasks.splice(index, 1);
                            this.activeTabIndex = 0;
                            break;
                        case "Delete Forever":
                            this.completedTasks.splice(index, 1);
                            break;
                        case "Cancel" || undefined:
                            break;
                    }
                });
            },

            onCompletedCircleTap: function(todo) {
                const index = this.completedTasks.indexOf(todo);
                this.todos.unshift(todo);
                this.completedTasks.splice(index, 1);
                this.activeTabIndex = 0;
            },

            onTabChange: function(tab) {
                this.activeTabIndex = tab.value;
                console.log(tab.value);
            },
            onReturnPress: function() {
                if (this.task.trim() === "") {
                    this.$refs.textInput.nativeView.focus();
                    return;
                }
                this.todos.unshift({
                    name: this.task
                });
                this.task = "";
            }
        },

        data() {
            return {
                todoIcon: platform.isIOS ? "~/./images/To-Do@3x.png" : "",
                completedIcon: platform.isIOS ? "~/./images/Completed@3x.png" :
                    "",
                todos: [{
                    name: "Goto Store"
                }],
                completedTasks: [],
                task: "",
                activeTabIndex: 0,
                message: "Hello there you all niggas!"
            };
        }
    };
</script>

<style scoped>
    ActionBar {
        background-color: #35495E;
        color: #fff
    }

    .header {
        background-color: #35495E;
        color: white;
        font-size: 34;
        font-weight: 600;
        padding: 0 15 15 15;
        margin: 0;
    }

    .list-entry {
        padding: 0 15;
        color: #42B883;
    }

    .circle {
        width: 30;
        height: 30;
        padding: 0;
        color: #42B883;
        font-size: 25;
        border-color: #42B883;
        border-width: 2;
        border-radius: 50;
    }

    .list-entry .circle {
        margin: 0 10 0 0;
    }

    .list-entry-completed .circle {
        color: white;
        background-color: #42B883;
        text-align: center;
        padding: 0;
    }

    .list-entry Label {
        font-weight: bold;
        font-size: 17;
        vertical-align: middle;
        padding: 17 0;
        margin: 0;
    }

    .completed {
        background-color: #42B883;
    }

    TextField {
        width: 100%;
        font-size: 17;
        color: black;
        placeholder-color: #C1C1C1;
        padding: 17;
        border-width: 0 0 1 0;
        border-color: #E0E0E0;
    }
</style>