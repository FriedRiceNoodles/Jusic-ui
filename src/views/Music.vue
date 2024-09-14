<template>
  <div>
    <div v-if="isPlay">
      <navigation :musichouse="musichouse" @openShareDialog="openShare = !openShare"></navigation>
     
      <mu-container class="demo-container">
             <mu-row style="margin-bottom: 23px;"></mu-row> 
        <mu-row gutter>
          <mu-col span="12" sm="12" md="8" lg="8" xl="9">
            <mu-col span="12">
              <mu-row>
                <mu-col
                  span="12"
                  sm="12"
                  md="5"
                  lg="4"
                  xl="3"
                  style="text-align: center; padding: 0 0 40px 0;" @click="playMusic"
                >
               
                  <mu-avatar
                    :size="albumRotateSize"
                    style="border: 2px solid rgba(26, 26, 26, 0.5); overflow: hidden; box-shadow: inset 0 0 20px 2px #000;"
                  >
                    <img
                      :class="albumRotate ? 'album album-rotate' : 'album'"
                      :src="
                        music.pictureUrl
                          ? music.pictureUrl
                          : require('../assets/images/logo.png')
                      "
                      :style="albumRotateStyle"
                      alt="pic"
                    />
                  </mu-avatar>
                </mu-col>
                <mu-col span="12" sm="12" md="7" lg="8" xl="9">
                  <div
                    style="font-size: 24px; font-weight: 400; margin: 4px 0 10px 0; min-height: 31px;"
                  >{{ music ? music.name : "" }}
                   <mu-button flat color="teal" style="float:right;" @click="searchTop">
                    <mu-icon left value="wb_sunny"></mu-icon>
                      热歌榜
                    </mu-button>
                    <mu-button flat color="teal" style="float:right;" @click="openBotttomPickHistorySheet">
                    <mu-icon left value="history"></mu-icon>
                      点歌历史
                    </mu-button>
                    
                  </div>
                  <div style="font-size: 16px; font-weight: 400; margin: 10px 0; min-height: 21px;">
                    专辑: &nbsp;{{
                    music.album ? "《" + music.album.name + "》" : ""
                    }}
                    歌手: &nbsp;{{ music ? music.artist : "" }}
                  </div>

                  <div
                    style="dfont-size: 14px; font-weight: 400; margin: 50px 0 10px 0; min-height: 21px;"
                  ><span style="color:orange;cursor: pointer" v-if="openLyrics" @click="openLyrics=!openLyrics">  ↑↑↑  </span><span style="color:#009688;cursor: pointer" @click="openLyrics=!openLyrics" v-else>  ↓↓↓  </span>{{ lyric }}</div>
                  <small id="musicEndTime" style="float: right">
                    {{
                    playerTime
                    }}
                  </small>
                  <mu-linear-progress mode="determinate" :value="progress" color="#009688"></mu-linear-progress>
                  <mu-flex justify-content="center">
                    <mu-flex class="flex-demo" justify-content="start"><mu-icon value="volume_up" color="teal"></mu-icon></mu-flex>
                    <mu-flex class="flex-demo" justify-content="start" fill> <mu-slider
                    class="demo-slider"
                    color="#009688"
                    v-model="volume"
                    style="color: rgb(0, 150, 136);"
                  ></mu-slider></mu-flex>
                  </mu-flex>      
                </mu-col>
              </mu-row>
            </mu-col>
           
            <mu-col span="12">
               <div style="margin-bottom:10px;height:250px" v-if="openLyrics">
                    <Lyrics :lyrics="lyrics" :currentTime="currentTime"/>
            </div>
            <div>
                            <VEmojiPicker @select="onSelectEmoji" :dark="true" style="width:100%;height:100px;"/>

            </div>
                <mu-data-table
                style="background-color: transparent;max-height:380px;overflow:auto;"
                :selectable="false"
                :hover="false"
                :columns="columns"
                :data="pick"
              >
                <template slot-scope="scope">
                  <td class="is-left">
                     <a @click="removeCollect(scope.row)" v-if="favoriteMap[scope.row.id] != null && favoriteMap[scope.row.id] != undefined">
                      <mu-icon value="favorite" size="20" color="red"></mu-icon>
                    </a>   
                     <a @click="collectMusic(scope.row)" v-else>
                      <mu-icon value="favorite" size="20" color="white"></mu-icon>
                    </a>   
                    {{ scope.$index + 1 }} 
                  
                  </td>
                  <td class="is-left">       
                   <a @click="goodMusic(scope.row)"
                      v-if="scope.$index != 0 && good">
                    <mu-icon value="thumb_up" size="20" color="teal"></mu-icon>
                  </a>
                    {{
                    isRoot || isAdmin
                    ? scope.row.name + `[${scope.row.id}]`
                    : scope.row.name
                    }}
                  </td>
                  <td class="is-center">{{ scope.row.artist }}</td>
                  <td class="is-center">{{ "《" + scope.row.album.name + "》" }}</td>
                  <!--                                    <td class="is-center">{{scope.row.nickName + scope.row.sessionId?`[${scope.row.sessionId}]`: '[]'}}</td>-->

                  <td class="is-center">
                    {{
                    isRoot || isAdmin
                    ? scope.row.nickName +
                    (scope.row.sessionId
                    ? `[${scope.row.sessionId}]`
                    : "")
                    : scope.row.nickName
                    }}
                  </td>
                </template>
              </mu-data-table>
              
            </mu-tabs>
            
            </mu-col>
          </mu-col>
          <mu-col span="12" sm="12" md="4" lg="4" xl="3">
            <mu-col
              :style="
                screenWidth < 766 && screenWidth !== 0
                  ? 'margin: 60px 0 200px 0;'
                  : ''
              "
            >
            <mu-flex justify-content="center" style="margin-bottom:10px;">
              <mu-button round color="transparent" @click="openHouse = !openHouse">
                    <mu-icon left value="account_balance"></mu-icon>
                        听歌房
              </mu-button> 
              </mu-flex>
              <div style="font-size: 16px; font-weight: 400;">
                 <mu-button flat color="white"  @click="houseUser">
                  <mu-icon left value="supervisor_account"></mu-icon>
                      {{online}}
                </mu-button >
                  <mu-button flat color="white"  @click="clearScr" style="float:right;">
                  <mu-icon left value="clear_all"></mu-icon>
                      
                </mu-button>
              
              </div>
              <div id="chat-container">
                <div
                  v-for="(item, index) in chatData"
                  :style="item.type === 'notice' ? 'text-align: center' : ''"
                  style="padding: 10px 0"
                >
                  <div>
                    <small class="chat-data-user">
                      {{
                      (isRoot || isAdmin) && item.type === "chat"
                      ? item.nickName + `[${item.sessionId}]`
                      : item.nickName
                      }}{{item.sendTime?"  "+formatterFullTime(item.sendTime):''}}
                    </small>
                  </div>
                  <div v-if="item.type === 'notice'">
                    <span class="chat-data-notice">{{ item.content }}</span>
                  </div>
                  <div v-if="item.type === 'chat'" class="chat-data-content">
                    <span>{{ item.content }}</span>
                    <img
                      v-if="item.images.length > 0"
                      v-for="(img, index) in item.images"
                      :src="img"
                      alt
                      style="width: 100%; display: block"
                    />
                  </div>
                </div>
              </div>
              <div
                :class="
                  screenWidth < 766 && screenWidth !== 0
                    ? 'message-input-group'
                    : ''
                "
              >
                <mu-text-field
                  id="chatInput"
                  :value="chatMessage"
                  @input="updateChatMessage"
                  @keydown="messageFieldEnterHandler"
                  placeholder="Message..."
                  color="primary"
                  class="width-size-100 chat-message"
                ></mu-text-field>
                <br />
                <div style="color:white;" >
                  <mu-radio :value="'wy'" v-model="sourceChat" color="primary" :label="'网易'" class="searchSource"></mu-radio>
                  <mu-radio :value="'qq'" v-model="sourceChat" color="primary" :label="'QQ'" class="searchSource"></mu-radio>
                  
                   <mu-button flat color="primary" @click="openManual = !openManual"><mu-icon left value="assignment"></mu-icon>教程及直播
                  </mu-button>               
             </div>

                  <mu-flex class="flex-wrapper" align-items="center">

                <mu-button
                  v-if="!isContented"
                  @click="connect"
                  color="primary"
                  style="width: 90%"
                >连接服务器</mu-button>
                <mu-button
                  v-if="isContented"
                  @click="sendHandler"
                  color="primary"
                  style="width: 80%"
                >发送消息</mu-button>
                 <mu-button icon @click="openBotttomSheet" style="font-size:20px">😃  
              </mu-button>
                <mu-button icon @click="openBotttomSheet" >
                    <mu-icon value="favorite" color="red"></mu-icon>
                </mu-button>
                </mu-flex>
              
                  <div style="padding-top: 10px;">
               
                  <mu-chip style="margin-right:10px;"
                    color="rgba(0, 150, 136, 0.5)"
                    @click="openPictureSearch = !openPictureSearch"
                  >斗图</mu-chip>
                  <mu-chip style="margin-right:10px;"  color="rgba(0, 150, 136, 0.5)" @click="musicSkipVote">切歌</mu-chip>
                  <mu-chip style="margin-right:10px;" color="rgba(0, 150, 136, 0.5)" @click="openSearch = !openSearch">点歌</mu-chip>
                  <mu-chip style="margin-right:10px;"  color="rgba(0, 150, 136, 0.5)" @click="openSearchGd = !openSearchGd">歌单</mu-chip>
             
             </div>
              </div>
            
            </mu-col>
           
          </mu-col>
        </mu-row>
      </mu-container>
      <div id="blur" style="opacity: 0.4">
        <img :src="music.pictureUrl" alt="blur-img" />
      </div>
      <div>
        <audio rel="noreferrer"
          id="music"
          :src="music.url"
          @timeupdate="musicTimeUpdate"
          controls
          autoplay="autoplay"
          @canplaythrough="nextSong"
          @canplay="setCurrentTime"
          style="display: none"
        ></audio>
        <!-- <audio id="music2" :src="music2.url" style="display: none"></audio> -->
      </div>
    </div>
    <div id="play" v-if="!isPlay" :style="backgroundDiv" >
    <mu-flex class="flex-wrapper" align-items="center">
       <mu-flex class="flex-wrapper" justify-content="start" fill style="margin-top:10px;">
      <mu-button color="info" flat @click="linkDownload('https://tx.alang.run/sponsor')">
       <mu-icon left value="favorite"></mu-icon>
          赞赏</mu-button>
    </mu-flex>
    <mu-flex class="flex-wrapper" justify-content="end" fill style="margin-top:10px;">
      <mu-button color="info" flat @click="linkDownload('https://tx.alang.run/release')">
       <mu-icon left value="android"></mu-icon>
          APP</mu-button>
    </mu-flex>
    </mu-flex>
    <mu-flex class="flex-wrapper" align-items="center">
      <mu-flex class="flex-wrapper" justify-content="center" fill >
        <mu-text-field v-model="houseSearch" placeholder="房间搜索" style="width:150px"></mu-text-field>
        <mu-switch v-model="houseHide" color="primary" label="隐藏空房" style="padding-top:4px"></mu-switch>   
      </mu-flex> 
    </mu-flex>
    <mu-flex class="flex-wrapper" justify-content="center" style="margin-top:10px;" wrap="wrap">
      <mu-flex  v-for="house, index in filteredHomeHouses" :key="house.id" @click="enterHomeHouse(house.id,house.name,house.needPwd)">
          <mu-tooltip  :content="house.desc">

           <mu-badge :content="house.population?house.population+'':'0'" circle :color="house.population>0?'info':''"  style="margin:8px 7px;" class="demo-icon-badge">
            <mu-chip class="demo-chip" color="teal" >
              <mu-avatar :size="32" :color="house.needPwd?'blue300':'green'">
                <mu-icon :value="house.needPwd?'lock':'lock_open'"></mu-icon>
              </mu-avatar>
              {{house.name}}
            </mu-chip>
           </mu-badge>
          </mu-tooltip>
      </mu-flex>
        
    </mu-flex>

      <mu-flex class="flex-wrapper" justify-content="center" style="padding-top:30px;">
       <mu-form :model="homeHouse" class="mu-demo-form" align="center" >
         <div align="center">
            <mu-text-field v-model="homeHouse.name" placeholder="房间名称"></mu-text-field>
          <mu-text-field v-model="homeHouse.desc" placeholder="房间描述"></mu-text-field>
         <mu-text-field
          v-if="homeHouse.needPwd"
          placeholder="房间密码"
          v-model="homeHouse.password"
          :action-icon="visibility ? 'visibility_off' : 'visibility'"
          :action-click="() => (visibility = !visibility)"
          :type="visibility ? 'text' : 'password'"
          ></mu-text-field>
            <mu-text-field  action-icon="favorite"
          :action-click="() => (linkDownload('https://tx.alang.run/sponsor'))" v-if="homeHouse.enableStatus" v-model="homeHouse.retainKey" placeholder="赞赏200大洋订单号"></mu-text-field>
          
            </div>
        <mu-flex class="flex-wrapper" align-items="center">
                    <mu-flex class="flex-demo" justify-content="end" fill> <mu-switch v-model="homeHouse.needPwd" color="primary" label="房间密码"></mu-switch></mu-flex>     
          <mu-flex class="flex-demo" justify-content="center"><mu-button color="primary" @click="createHomeHouse">创建房间</mu-button></mu-flex>
                    <mu-flex class="flex-demo" justify-content="start" fill> <mu-switch v-model="homeHouse.enableStatus" color="primary" label="房间永存"></mu-switch></mu-flex>     
        </mu-flex>
      </mu-form>
      </mu-flex>
      <mu-dialog id="sharereach"  width="100%"
      transition="slide-bottom"
      fullscreen :open.sync="openShareReach">
       <mu-appbar color="primary" :title="houseReachName">
        <mu-button slot="right" flat @click="openShareReach = !openShareReach">X</mu-button>
      </mu-appbar>
       <mu-icon @click="reachHouse" value="play_circle_filled" color="primary" size="150"
                     style="position: fixed; top: calc(50% - 75px); left: calc(50% - 75px); cursor: pointer;"></mu-icon>
    </mu-dialog>
    </div>
    <mu-dialog id="search" width="100%"
      transition="slide-bottom"
      fullscreen  :open.sync="openSearch">
      <mu-appbar color="primary" title="歌曲搜索">
        <mu-button slot="right" flat @click="openGd">歌单</mu-button>
        <mu-button slot="right" flat @click="closeGq">X</mu-button>
      </mu-appbar>
      <mu-container style="width:100%;">
        <mu-row>
          <mu-col>
            <mu-text-field
              :value="searchKeyword"
              @input="updateSearchKeyword"
              @keydown.enter="search"
              :placeholder="placeHolderGq"
              color="#009688"
              class="width-size-100"
              style="text-align: center"
            ></mu-text-field>
            <mu-radio
              :value="'wy'"
              v-model="source"
              color="primary"
              :label="'网易'"
              class="searchradio"
            ></mu-radio>
            <mu-radio
              :value="'qq'"
              v-model="source"
              color="primary"
              :label="'QQ'"
              class="searchradio"
            ></mu-radio>
           <mu-radio
              :value="'wydt'"
              v-model="source"
              color="primary"
              :label="'电台'"
              class="searchradio"
            ></mu-radio>
            <mu-radio
              :value="'lz'"
              v-model="source"
              color="primary"
              :label="'禁歌'"
              class="searchradio"
            ></mu-radio>
            <mu-radio
              :value="'ai'"
              v-model="source"
              color="primary"
              :label="'AI'"
              class="searchradio"
            ></mu-radio>
            <a  @click="playCurrentPage">
                <mu-avatar size="23" slot="avatar" style="margin-left:3px;">
                    <img src="../assets/images/play.png" title="播放当前页所有歌曲"/>
                  </mu-avatar>
                </a>
          </mu-col>
          <mu-col span="1">
            <mu-button class="search_btn" icon @click="search">
              <mu-icon value="search"></mu-icon>
            </mu-button>
          </mu-col>
        </mu-row>
        <mu-row>
          <mu-data-table
            style="background-color: transparent"
            :selectable="false"
            :hover="false"
            :columns="searchColumns"
            :data="searchData"
          >
            <template slot-scope="scope">
              <td class="is-left">{{ scope.$index + 1 }}</td>
              <!--<td class="is-center">-->
              <!--<a v-if="showPickButton(scope.row.privilege)" class="search_pick_btn" @click="pickMusic(scope.row)">点歌</a>-->
              <!--<mu-tooltip v-if="!showPickButton(scope.row.privilege)" content="当前音乐不能点播">-->
              <!--<a v-if="" class="search_pick_btn_disable">点歌</a>-->
              <!--</mu-tooltip>-->
              <!--</td>-->
              <td class="is-left">
                <a v-if="showPickButton(scope.row.privilege)" @click="pickMusic(scope.row)">
                  <mu-avatar size="20" slot="avatar">
                    <img src="../assets/images/play.png" />
                  </mu-avatar>
                </a>
                &nbsp;&nbsp;
                <a v-if="showPickButton(scope.row.privilege)" @click="pickMusic(scope.row,'flac')">
                  <mu-avatar size="20" slot="avatar">
                    <img src="../assets/images/hifi.png" />
                  </mu-avatar>
                </a>
                <mu-avatar size="20" slot="avatar" v-if="!showPickButton(scope.row.privilege)">
                  <mu-tooltip content="当前音乐不能点播">
                    <img src="../assets/images/noplay.png" />
                  </mu-tooltip>
                </mu-avatar>
                {{ scope.row.name }}
              </td>
              <td class="is-center">{{ scope.row.artist }}</td>
           <!--    <td  class="is-center">
                 <mu-card-media width="50px" heigth="50px">
                    <img :src="scope.row.picture_url">
                </mu-card-media>
              </td>-->
              <td class="is-center">{{ "《" + scope.row.album.name + "》" }}</td>
              <td class="is-center">{{ formatterTime(scope.row.duration / 1000) }}</td>
            </template>
          </mu-data-table>
        </mu-row>
        <mu-row>
          <mu-flex justify-content="center">
            <mu-pagination
              :total="searchCount"
              :current.sync="current"
              :page-count="pageCountCompute"
              :page-size="limit"
              @change="paginationChange"
            ></mu-pagination>
          </mu-flex>
        </mu-row>
      </mu-container>
    </mu-dialog>
     <mu-dialog id="searchGd"  width="100%"
      transition="slide-bottom"
      fullscreen :open.sync="openSearchGd">
       <mu-appbar color="primary" title="歌单搜索">
        <mu-button slot="right" flat @click="openUser">用户</mu-button>
        <mu-button slot="right" flat @click="openGq">歌曲</mu-button>
        <mu-button slot="right" flat @click="closeGd">X</mu-button>
      </mu-appbar>
      <mu-container style="width:100%;">
        <mu-row>
          <mu-col >
            <mu-text-field
              :value="searchKeywordGd"
              @input="updateSearchKeywordGd"
              @keydown.enter="searchGd"
              :placeholder="placeHolderGd"
              color="#009688"
              class="width-size-100"
              style="text-align: center"
            ></mu-text-field>
                <mu-radio
              :value="'wy'"
              v-model="sourceGd"
              color="primary"
              :label="'网易'"
              class="searchradio2"
            ></mu-radio>
              <mu-radio
              :value="'wy_user'"
              v-model="sourceGd"
              color="primary"
              :label="'用户'"
              class="searchradio2"
            ></mu-radio>
             <mu-radio
              :value="'wy_userdj'"
              v-model="sourceGd"
              color="primary"
              :label="'电台'"
              class="searchradio2"
            ></mu-radio>
            <mu-radio
              :value="'qq'"
              v-model="sourceGd"
              color="primary"
              :label="'QQ'"
              class="searchradio"
            ></mu-radio>
            <mu-radio
              :value="'qq_user'"
              v-model="sourceGd"
              color="primary"
              :label="'用户'"
              class="searchradio"
            ></mu-radio>
          </mu-col>
          <mu-col span="1">
            <mu-button class="search_btn" icon @click="searchGd">
              <mu-icon value="search"></mu-icon>
            </mu-button>
          </mu-col>
        </mu-row>
        <mu-row>
          <mu-data-table
            style="background-color: transparent"
            :selectable="false"
            :hover="false"
            :columns="searchColumnsGd"
            :data="searchDataGd"
          >
            <template slot-scope="scope2">
              <!--<td class="is-center">-->
              <!--<a v-if="showPickButton(scope.row.privilege)" class="search_pick_btn" @click="pickMusic(scope.row)">点歌</a>-->
              <!--<mu-tooltip v-if="!showPickButton(scope.row.privilege)" content="当前音乐不能点播">-->
              <!--<a v-if="" class="search_pick_btn_disable">点歌</a>-->
              <!--</mu-tooltip>-->
              <!--</td>-->
              <td class="is-left">
                {{ scope2.$index + 1 }}.
                <a  @click="songlistDetail(scope2.row)">
                <mu-icon :value="'reply'"></mu-icon>
                </a>
                {{ scope2.row.name }}
              </td>
              <td>
                <mu-card-media width="50px" heigth="50px">
                    <img :src="scope2.row.pictureUrl">
                </mu-card-media>
              </td>
              <td class="is-center">{{ scope2.row.desc|ellipsis }}</td>

              <td class="is-center">{{ scope2.row.creator }}</td>
              <td class="is-center">{{ scope2.row.creatorUid }}</td>
              <td class="is-center">{{ scope2.row.id }}</td>
               <td class="is-center">{{ scope2.row.playCount }}</td>
               <td class="is-center">{{ scope2.row.songCount }}</td>
            </template>
          </mu-data-table>
        </mu-row>
        <mu-row>
          <mu-flex justify-content="center">
            <mu-pagination
              :total="searchCountGd"
              :current.sync="currentGd"
              :page-count="pageCountCompute"
              :page-size="limit"
              @change="paginationChangeGd"
            ></mu-pagination>
          </mu-flex>
        </mu-row>
      </mu-container>
    </mu-dialog>
    <mu-dialog id="searchUser"  width="100%"
      transition="slide-bottom"
      fullscreen :open.sync="openSearchUser">
       <mu-appbar color="primary" title="用户搜索">
        <mu-button slot="right" flat @click="openGdFromUser">歌单</mu-button>
        <mu-button slot="right" flat @click="closeUser">X</mu-button>
      </mu-appbar>
      <mu-container style="width:100%;">
        <mu-row>
          <mu-col >
            <mu-text-field
              :value="searchKeywordUser"
              @input="updateSearchKeywordUser"
              @keydown.enter="searchUser"
              placeholder="请输入用户昵称"
              color="#009688"
              class="width-size-100"
              style="text-align: center"
            ></mu-text-field>
            <mu-radio
              :value="'wy'"
              v-model="sourceUser"
              color="primary"
              :label="'网易'"
              class="searchradio"
            ></mu-radio>
          </mu-col>
          <mu-col span="1">
            <mu-button class="search_btn" icon @click="searchUser">
              <mu-icon value="search"></mu-icon>
            </mu-button>
          </mu-col>
        </mu-row>
        <mu-row>
          <mu-data-table
            style="background-color: transparent"
            :selectable="false"
            :hover="false"
            :columns="searchColumnsUser"
            :data="searchDataUser"
          >
            <template slot-scope="scope3">
              <!--<td class="is-center">-->
              <!--<a v-if="showPickButton(scope.row.privilege)" class="search_pick_btn" @click="pickMusic(scope.row)">点歌</a>-->
              <!--<mu-tooltip v-if="!showPickButton(scope.row.privilege)" content="当前音乐不能点播">-->
              <!--<a v-if="" class="search_pick_btn_disable">点歌</a>-->
              <!--</mu-tooltip>-->
              <!--</td>-->
              <td class="is-left">
                {{ scope3.$index + 1 }}.
                <a  @click="songtableDetail(scope3.row)">
                <mu-icon :value="'reply'"></mu-icon>
                </a>
                {{ scope3.row.nickname }}
                 <a  @click="djtableDetail(scope3.row)" >
                   <font style="color:orange !important;">电台</font>
                </a>
              </td>
              <td class="is-center">
                <mu-avatar size="36">
                <img :src="scope3.row.avatarUrl">
              </mu-avatar>
              </td>
              <td class="is-center">{{ scope3.row.userId }}</td>

              <td class="is-center">{{ scope3.row.signature |ellipsis }}</td>
              <td class="is-center">{{ scope3.row.description|ellipsis}}</td>
              <td class="is-center">{{ scope3.row.gender==1?'男':(scope3.row.gender==2?'女':'未知' )}}</td>

            </template>
          </mu-data-table>
        </mu-row>
        <mu-row>
          <mu-flex justify-content="center">
            <mu-pagination
              :total="searchCountUser"
              :current.sync="currentUser"
              :page-count="pageCountCompute"
              :page-size="limit"
              @change="paginationChangeUser"
            ></mu-pagination>
          </mu-flex>
        </mu-row>
      </mu-container>
    </mu-dialog>
     <mu-dialog id="share"  width="100%"
      transition="slide-bottom"
      fullscreen :open.sync="openShare">
       <mu-appbar color="primary" title="分享">
        <mu-button slot="right" flat @click="openShare = !openShare">X</mu-button>
      </mu-appbar>
       
    <mu-flex class="flex-wrapper" justify-content="center">
          <mu-card style="max-width: 375px;margin-top: 20px;"  align="center">
          <mu-card-header :title="musichouse" :sub-title="homeDesc?(homeDesc):'做一个自带背景音乐的人'" align="left">
          <mu-avatar slot="avatar" size="50">
              <img src="../assets/images/logo.png">
          </mu-avatar>
          </mu-card-header>
          <mu-card-media style="width:250px">
          <img :src="miniQrcode"/>
        </mu-card-media>
        <mu-card-title sub-title="微信扫码穿梭到小程序"></mu-card-title>
        		 <!-- <mu-divider></mu-divider> -->

          <mu-card-media style="margin-top:10px;">
          <qrcode-vue
	          id="qrcodeBox"
            level="H"
	          :size="qrcodeVue.size"
	          :value="qrcodeVue.value"
	          :background="qrcodeVue.bgColor"
	          :foreground="qrcodeVue.fgColor">
         </qrcode-vue>
        </mu-card-media>
       <mu-card-title title="分享链接" :sub-title="qrcodeVue.value"></mu-card-title>
      </mu-card>
    </mu-flex>
    </mu-dialog>
    <mu-dialog id="search-picture" width="auto" :open.sync="openPictureSearch">
      <chat-search-picture></chat-search-picture>
    </mu-dialog>
    <mu-dialog
      id="house_dialog"
      width="100%"
      transition="slide-bottom"
      fullscreen
      :open.sync="openHouse">
      <mu-appbar color="primary" title="房间">
        <mu-button slot="right" flat @click="innerHouseHide=!innerHouseHide">{{innerHouseHide?'显示空房':'隐藏空房'}}</mu-button>
        <mu-button slot="right" flat @click="closeHouse">X</mu-button>
      </mu-appbar>
        <mu-flex class="flex-wrapper" justify-content="center">
            <mu-form :model="house" class="mu-demo-form" align="center">
             <div align="center">
              <mu-text-field v-model="house.name" placeholder="房间名称搜索"></mu-text-field>
              <mu-text-field v-model="house.desc" placeholder="房间描述"></mu-text-field>
              <mu-text-field v-if="house.needPwd" placeholder="房间密码" v-model="house.password" :action-icon="visibility ? 'visibility_off' : 'visibility'"
                :action-click="() => (visibility = !visibility)"
                :type="visibility ? 'text' : 'password'"
              ></mu-text-field>
               <mu-text-field action-icon="favorite" :action-click="() => (linkDownload('https://tx.alang.run/sponsor'))" v-if="house.enableStatus" v-model="house.retainKey" placeholder="赞赏200大洋订单号"></mu-text-field>
              </div>

              <mu-flex class="flex-wrapper" align-items="center">
                <mu-flex class="flex-demo" justify-content="end" fill> <mu-switch v-model="house.needPwd" color="primary" label="房间密码"></mu-switch></mu-flex>     
                <mu-flex class="flex-demo" justify-content="start" ><mu-button color="primary" @click="createHouse">创建房间</mu-button></mu-flex>
                <mu-flex class="flex-demo" justify-content="start" fill> <mu-switch v-model="house.enableStatus" color="primary" label="房间永存"></mu-switch></mu-flex>     
              </mu-flex>
            </mu-form>
        </mu-flex>
      <mu-flex class="flex-wrapper" justify-content="center" style="padding-top:30px;" wrap="wrap"	>
        <mu-flex   v-for="houseItem, index in filteredHouses"
          :key="houseItem.id"
          @click="enterHouse(houseItem.id,houseItem.name,houseItem.needPwd)" >
           <mu-tooltip :content="houseItem.desc">
            <mu-badge :content="houseItem.population?houseItem.population+'':'0'" circle :color="houseItem.population>0?'info':''"   style="margin:8px 7px;" class="demo-icon-badge">
            <mu-chip class="demo-chip" color="teal">
              <mu-avatar :size="32" :color="houseItem.needPwd?'blue300':'green'">
                <mu-icon :value="houseItem.needPwd?'lock':'lock_open'"></mu-icon>
              </mu-avatar>
              {{houseItem.name}}
            </mu-chip>
            </mu-badge>
          </mu-tooltip>
        
        </mu-flex>
         
      </mu-flex>
    </mu-dialog>
     <mu-drawer width="300" :open.sync="openManual" :docked="false" :right="true">
            <mu-card style="width: 100%; max-width: 375px; margin: 0 auto;">
                <mu-card-header title="使用教程" sub-title="欢迎加群1029454474交流">
                    <mu-avatar size="45" slot="avatar">
                        <img style="border-radius: 50%; border: 1px solid rgba(255, 255, 255, 0.2);" src="../assets/images/logo.png">
                    </mu-avatar>
                </mu-card-header>
              
                <mu-card-text>
                  声明：本网站仅供学习交流。
                </mu-card-text>
            </mu-card>
            <mu-card>
                <BiliLive :playingId="playingId"/>
            </mu-card>
            <mu-card-title title="用户"  sub-title="以下命令在聊天框输入"></mu-card-title>
            <mu-card-text>
                <p>1.输入 “点歌   歌名” 即可点歌。例如：点歌 春夏秋冬，支持输入网易云音乐 ID 点歌。</p>
                <br/>
                 <p>
                  2.没有想要点的音乐？ 请点击--> “
                  <span
                    @click="openManual = !openManual;openSearch = !openSearch"
                    style="cursor: pointer; color: #009688;"
                  >[点歌]</span>“，如果知道歌单id，还可以在歌曲窗口直接加*搜索： <span style="color: #009688;">*歌单id</span><br/>
                </p>
                <br/>
                <p>
                  3.不知道歌单id?,请点击--> 
                  <span
                    @click="openManual = !openManual;openSearchGd = !openSearchGd"
                    style="cursor: pointer; color: #009688;"
                  >[歌单]</span>提示：歌单页面可以搜索网易歌单、网易用户id的歌单、qq歌单、qq用户id的歌单
                </p>
                                <br />

                 <p>
                  4.如点错歌曲可以输入 “
                  <span style="color: #009688;">删除音乐 歌名</span>” 即可删除歌曲，管理员可以使用歌曲id删除。
                </p>
                <br />
                <p>
                  5.如遇不好听的歌可以输入 “
                  <span style="color: #009688;">投票切歌</span>” 或者点击 “
                  <span @click="musicSkipVote" style="cursor: pointer; color: #009688;">[切歌]</span>”，默认当投票人数大于在线人数 30% 时将会切歌。管理员可以设置切歌率。
                </p>
                <br />
                <p>
                  6.输入 “
                  <span style="color: #009688;">设置昵称 名字</span>”
                   可以设置自己的显示昵称，仅限当前客户端有效。
                </p>
                                <br />
                <p>
                  7.私聊：输入 “
                  <span style="color: #009688;">@用户id 内容</span>”
                   可以私聊相应用户，用户id即用户ip后面那一串字母，如ju2etxv2。
                   不知道用户id,试着点击在线人数图标。
                </p>
                                <br />

                <p>
                  8.想要斗图？ ┏ (゜ω゜)=☞ “
                  <span
                    @click="openManual = !openManual;openPictureSearch = !openPictureSearch"
                    style="cursor: pointer; color: #009688;"
                  >[斗图]</span>”
                </p>
                <br/>
                 <p>
                  9.倒计时退出房间
                   输入 “
                  <span style="color: #009688;">倒计时退出 1</span>”
                  则将在1分钟后退出房间。取消倒计时退出：<span style="color: #009688;">取消退出</span>”
                </p>
                <br/>
                 <p>
                  10.如果有什么好的想法、建议或问题可以单项向管理员发送消息，（＾∀＾●）ﾉｼ
                  “
                  <span style="color: #009688;">@管理员 内容</span>”,
                  空格隔开哦!
                </p>
                                <br />

                <p>另外也可以在项目仓库开个 issue 进行公开讨论</p>
                 </mu-card-text>
           
        <mu-divider></mu-divider>
          <mu-card-title title="管理员"  sub-title="以下命令在聊天框输入"></mu-card-title>
            <mu-card-text>
                 <p>
                  1.登录： “
                  <span style="color: #009688;">admin 123456</span>” 。
                 </p>
                 <br/>
                 <p>
                  2.修改密码： “
                  <span style="color: #009688;">修改密码 654321</span>” 。
                </p>
                <br/>
                 <p>
                  3.管理员公告 “
                  <span style="color: #009688;">公告 请文明聊天</span>”。
                </p><br/>
                <p>
                  4.点赞模式（歌曲列表按点赞数排序）： “
                  <span style="color: #009688;">点赞模式</span>” 退出则“
                  <span style="color: #009688;">退出点赞模式</span>” 。
                </p><br/>
                <p>
                  5.随机模式（歌曲列表随机播放）： “
                  <span style="color: #009688;">随机模式</span>” 退出则“
                  <span style="color: #009688;">退出随机模式</span>” 。
                </p><br/>
                <p>
                  6.修改投票切歌率： “
                  <span style="color: #009688;">投票切歌率 1</span>” 数值在(0,1]。如：设置成0.5则表示房间人数一半赞同即可切歌。</p><br/>
                <p>
                  7.禁止切歌：“
                  <span style="color: #009688;">禁止切歌</span>” 启用则“
                  <span style="color: #009688;">启用切歌</span>” 。</p><br/>
                <p>
                  8.禁止点歌：“
                  <span style="color: #009688;">禁止点歌</span>” 启用则“
                  <span style="color: #009688;">启用点歌</span>” 。</p><br/>
                <p>
                  9.清空列表：“
                  <span style="color: #009688;">清空列表</span>” 。
                </p><br/>
                <p>
                  10.清空默认播放列表：“
                  <span style="color: #009688;">清空默认列表</span>” 。</p><br/>
                <p> 11.设置默认播放列表（当点歌列表为空时，默认从此加载歌曲）：“
                  <span style="color: #009688;">设置默认列表 24381616,23828282</span>” ，其中243881616和23828282分别是歌单id</p><br/>
                <p> 12.默认列表歌曲数：“
                  <span style="color: #009688;">默认列表歌曲数</span>” 。</p><br/>
                <p>13.置顶音乐： “
                  <span style="color: #009688;">置顶音乐 音乐id</span>” 音乐id即歌曲列表中歌曲后面那一串字母，如411214279。</p><br/>
                <p>14.拉黑音乐：“
                  <span style="color: #009688;">拉黑音乐 音乐id</span>” 漂白则“
                  <span style="color: #009688;">漂白音乐 音乐id</span>” 。</p><br/>
                <p>15.音乐黑名单： “
                  <span style="color: #009688;">音乐黑名单</span>” 。</p><br/>
                <p>16.拉黑用户：“
                  <span style="color: #009688;">拉黑用户 用户id</span>” 漂白则“
                  <span style="color: #009688;">漂白用户 用户id</span>” 用户id即用户ip后面那一串字母，如ju2etxv2。</p><br/>
                <p>17.用户黑名单： “
                  <span style="color: #009688;">用户黑名单</span>” 。</p><br/>
                <p>18.设置点歌人：“
                  <span style="color: #009688;">设置点歌人 用户id</span>” 用户id即用户ip后面那一串字母，如ju2etxv2。取消则“
                  <span style="color: #009688;">取消点歌人 用户id</span>” 。</p><br/>
                <p>19.设置切歌人：“
                  <span style="color: #009688;">设置切歌人 用户id</span>” 用户id即用户ip后面那一串字母，如ju2etxv2。取消则“
                  <span style="color: #009688;">取消切歌人 用户id</span>” 。
                </p>
                <br />
                 <p>
                  20.单曲循环： “
                  <span style="color: #009688;">单曲循环</span>” 退出则“
                  <span style="color: #009688;">退出单曲循环</span>” 。
                </p>
                 <br />
                 <p>
                  21.列表循环： “
                  <span style="color: #009688;">列表循环</span>” 退出则“
                  <span style="color: #009688;">退出列表循环</span>” 。
                </p><br/>
                 </mu-card-text>
           
        </mu-drawer>

        <mu-bottom-sheet class="sheet" :open.sync="open" style="max-height:380px;overflow:auto;">
    <mu-list>
      <mu-sub-header>
        <mu-button flat color="primary" @click="playAll">
              全部播放
            </mu-button>           
            <mu-button flat color="primary" @click="exportCollect">
              导出
            </mu-button>
            <mu-button flat color="primary" @click="importCollect">
              导入
            </mu-button>
             <input ref="fileInput" type="file" accept="application/json" style="display: none" @change="handleFileInputChange">
            <mu-button flat color="primary" @click="removeAllCollect">
              取消收藏
            </mu-button>
      </mu-sub-header>
  
      <mu-list-item v-for="(item,index) in sortCollect" :key="item.key">
       <mu-list-item-action @click="removeCollect(item.value)" style="width:10%;">
          <mu-icon value="favorite" color="red"></mu-icon>
        </mu-list-item-action>
         <mu-list-item-action @click="pickMusicNoToast(item.value,'320k')"  style="width:10%;">
          <mu-icon value="play_arrow" color="teal"></mu-icon>
        </mu-list-item-action>
         <mu-list-item-action @click="pickMusicNoToast(item.value,'flac')"  style="width:10%;">
           <mu-avatar size="20" slot="avatar">
                    <img src="../assets/images/hifi.png" />
                  </mu-avatar>
        </mu-list-item-action>
        <mu-list-item-title  style="width:80%;">{{index+1}}.{{item.value.name}}|{{item.value.artist}}|{{item.value.album.name}}</mu-list-item-title>
         
      </mu-list-item>
    </mu-list>
  </mu-bottom-sheet>
  <mu-bottom-sheet class="sheet" :open.sync="openPickHistory" style="max-height:380px;overflow:auto;">
    <mu-list>
      <mu-sub-header>
            <mu-button flat color="primary" @click="removeAllPickHistory">
              清空历史
            </mu-button>
            <mu-button flat color="primary" @click="seeMyselfPickHistory">
              只看自己
            </mu-button>
            <mu-button flat color="primary" @click="seeAllPickHistory">
              看所有
            </mu-button>
            <mu-text-field v-model="pickSearch" placeholder="搜索歌名|歌手|点歌人" style="width:150px"></mu-text-field>

      </mu-sub-header>
  
      <mu-list-item v-for="(item,index) in filteredPickHistoryList" :key="item.key">
           <mu-list-item-action @click="removeCollect(item)" style="width:6%;" v-if="favoriteMap[item.id] != null && favoriteMap[item.id] != undefined">
          <mu-icon value="favorite" color="red"></mu-icon>
        </mu-list-item-action>
       <mu-list-item-action @click="collectMusic(item)" style="width:6%;" v-else >
          <mu-icon value="favorite" color="white"></mu-icon>
        </mu-list-item-action>
         <mu-list-item-action @click="pickMusicNoToast(item,'320k')"  style="width:7%;">
          <mu-icon value="play_arrow" color="teal"></mu-icon>
        </mu-list-item-action>
         <mu-list-item-action @click="pickMusicNoToast(item,'flac')"  style="width:7%;">
           <mu-avatar size="20" slot="avatar">
                    <img src="../assets/images/hifi.png" />
                  </mu-avatar>
        </mu-list-item-action>
        <mu-list-item-title  style="width:80%;">{{index+1}}.{{item.name}}|{{item.artist}}|{{item.album.name}}(<span style='color:teal'>{{item.nickName}}</span>{{formatDate(item.pickTime)}})</mu-list-item-title>
      </mu-list-item>
    </mu-list>
  </mu-bottom-sheet>
  </div>
</template>

<script>
import { mapGetters, mapMutations } from "vuex";
import SockJS from "sockjs-client";
import Stomp from "stompjs";
import { sendUtils, messageUtils, timeUtils, musicUtils } from "../utils";
import { baseUrl, isProduction,kuwoHttps } from "../config/environment";
import Navigation from "../components/Navigation";
import ChatSearchPicture from "../components/ChatSearchPicture";
import BiliLive from "../components/BiliLive";
import Lyrics from "../components/Lyrics";

import wx from 'weixin-js-sdk';
import QrcodeVue from "qrcode.vue";
import { balloons } from "balloons-js";
export default {
  name: "Music",
  components: {
    Navigation,
    ChatSearchPicture,
    QrcodeVue,
    BiliLive,
    Lyrics
  },
  filters: {
    ellipsis (value) {
      if (!value) return ''
      if (value.length > 20) {
        return value.slice(0,20) + '...'
      }
      return value
    }
  },
  computed: {
    pageCountCompute(){
      if(!this.pageCount || this.pageCount < 5){
        return 5
      }
      return this.pageCount % 2 == 0? this.pageCount+1:this.pageCount;
    },
    sortCollect(){
      // 将对象的键值对转换为数组
      const array = Object.entries(this.favoriteMap);

      // 使用 JavaScript 的 sort 方法对数组按 name 字段排序
      array.sort((a, b) => {
        return b[1].pickTime-a[1].pickTime;
      });

      // 将排序后的数组映射为包含键值对的对象
      return array.map(([key, value]) => {
        return { key, value };
      });
    },
    filteredHomeHouses() {
    return this.homeHouses.filter(house => {
      return house.name.toLowerCase().indexOf(this.houseSearch.toLowerCase()) !== -1 && (this.houseHide?house.population > 0:true);
    });
    },
    filteredPickHistoryList() {
    return this.pickHistoryList.filter(item => {
      return this.pickSearch?((item.name.toLowerCase().indexOf(this.pickSearch.toLowerCase()) !== -1) ||(item.artist.toLowerCase().indexOf(this.pickSearch.toLowerCase()) !== -1) || item.nickName.indexOf(this.pickSearch) != -1):true;
    });
    },
    filteredHouses() {
    return this.houses.filter(house => {
      return house.name.toLowerCase().indexOf(this.house.name.toLowerCase()) !== -1 && (this.innerHouseHide?house.population > 0:true);
    });
    },
    ...mapGetters({
      isContented: "getIsConnected",
      online: "getSocketOnline",
      chatMessage: "getChatMessage",
      chatData: "getChatData",
      music: "getPlayerMusic",
      progress: "getPlayerProgress",
      playerTime: "getPlayerTime",
      // volume: 'getPlayerVolume',
      pick: "getPlayerPick",
      lyric: "getPlayerLyric",
      isRoot: "isSocketRoot",
      isAdmin: "isSocketAdmin",
      good: "isSocketGood",
      circle: "isSocketCircle",
      searchKeyword: "getSearchKeyword",
      searchKeywordGd: "getSearchKeywordGd",
      searchKeywordUser: "getSearchKeywordUser",
      searchData: "getSearchData",
      searchDataGd: "getSearchDataGd",
      searchDataUser: "getSearchDataUser",
      searchCount: "getSearchCount",
      searchCountGd: "getSearchCountGd",
      searchCountUser: "getSearchCountUser",
      music2: "getMusic2"
    }),
    ...mapMutations({
      // volume: 'setPlayerVolume'
    }),
    volume: {
      get: function() {
        return this.$store.getters.getPlayerVolume;
      },
      set: function(value) {
        music.volume = Number(value) / 100;
        this.$store.commit("setPlayerVolume", value);
      }
    }
  },
  data: () => ({
    isPlay: false,
    columns: [
      { title: "ID", name: "id", width: 88, align: "left" },
      { title: "歌曲", name: "name", width: 200, align: "left" },
      { title: "歌手", name: "calories", align: "center" },
      { title: "专辑", name: "fat", align: "center" },
      { title: "点歌人", name: "carbs", align: "center" }
    ],
    albumRotate: false,
    screenWidth: document.documentElement.clientWidth,
    albumRotateSize: 300,
    albumRotateStyle: "",
    openSearch: false,
    openSearchGd: false,
    openSearchUser: false,
    openHouse: false,
    openManual:false,
    searchColumns: [
      { title: "ID", name: "id", width: 40, align: "left" },
      // {title: '操作', name: 'op', align: 'center'},
      { title: "歌曲", name: "name", width: 200, align: "left" },
      { title: "歌手", name: "artist", align: "center" },
    //  { title: "封面", name: "picture_url", align: "center" },
      { title: "专辑", name: "album", align: "center" },
      { title: "时长", name: "duration", align: "center" }
    ],
     searchColumnsGd: [
      // {title: '操作', name: 'op', align: 'center'},
      { title: "歌单", name: "name", width: 200, align: "left" },
      { title: "封面", name: "pictureUrl", align: "center" },
      { title: "描述", name: "desc", align: "center" },
      { title: "创建者", name: "creator", align: "center" },
      { title: "创建者id", name: "creatorUid", align: "center" },
      { title: "歌单id", name: "id", align: "center" },
      { title: "播放量", name: "playCount", align: "center" },
      { title: "曲数", name: "songCount", align: "center" }
    ],
     searchColumnsUser: [
      // {title: '操作', name: 'op', align: 'center'},
      { title: "昵称", name: "nickname", width: 200, align: "left" },
      { title: "头像", name: "avatarUrl", align: "center" },
      { title: "用户id", name: "userId", align: "center" },
      { title: "签名", name: "signature", align: "center" },
      { title: "描述", name: "description", align: "center" },
      { title: "性别", name: "gender", align: "center" }
    ],
    keyword: "",
    current: 1,
    currentGd: 1,
    currentUser:1,
    limit: 10,
    pageCount: 7,
    openPictureSearch: false,
    source: "wy",
    sourceGd: "wy",
    sourceChat: "wy",
    sourceUser: "wy",
    house: { name: "", desc: "", password: "", needPwd: false,enableStatus:false,retainKey:"" },
    homeHouse: { name: "", desc: "", password: "", needPwd: false,enableStatus:false,retainKey:""},
    secondUrl: "",
    firstLoaded: 0,
    houses: [],
    homeHouses: [],
    musichouse: "一起听歌吧",
    loading: {},
    houseForward: "",
    visibility: false,
    favorite: false,
    playingId:"",
    houseId:"",
    housePwd:"123",
    connectType:"",
    houseIdNoAction:"",
    housePwdNoAction:"123",
    connectTypeNoAction:"",
    placeHolderGd:'试下为空搜索下(*^__^*)',
    placeHolderGq:'请输入关键字搜索',
     backgroundDiv:{
       overflow:"auto",
       position:"fixed",
       top:0,
       left: 0,
        width:"100%",
        height:"100%",
        zoom: 1,
        background:'url(' + require('../assets/images/background.jpg') + ') no-repeat',
        "background-size":"100% 100%",
         "background-size": "cover",
         "-webkit-background-size": "cover",
        "-o-background-size": "cover"
    },
     qrcodeVue: {
        size: 250,
        bgColor: "#fff",
        fgColor: "#000",
        value: ""	//二维码地址
      },
      openShare:false,
      openShareReach:false,
      houseReachId:'',
      houseReachName:'直达房间',
      homeDesc:'',
      closeClock:null,
      announceToast:null,
      lastLyric:'',
      currentLyric:'',
      favoriteMap:{},
      pickHistoryList:[],
      open:false,
      openPickHistory:false,
      miniQrcode:'',
      currentTime:0,
      lyrics:{},
      openLyrics:false,
      houseSearch:'',
      houseHide:false,
      innerHouseHide:false,
      pickSearch:null
   } ),
  methods: {
    onSelectEmoji(emoji){
             let input = document.getElementById("chatInput");
        let startPos = input.selectionStart;
        let endPos = input.selectionEnd;
        let resultText = input.value.substring(0, startPos) + emoji.data + input.value.substring(endPos);
        input.value = resultText;
        input.focus()
        input.selectionStart = startPos + emoji.data.length
        input.selectionEnd = startPos + emoji.data.length
        this.$store.commit("setChatMessage", resultText);

    },
    formatDate(value) {
      if (value) {
        return new Date(value).toLocaleDateString('zh-CN', {
          year: 'numeric',
          month: '2-digit',
          day: '2-digit',
          hour: '2-digit',
          minute: '2-digit',
          second: '2-digit',
        });
      }
    },
    play: function() {
      this.getScreenWidth();
      this.isPlay = !this.isPlay;
      this.connect();
    },
    connect: function() {
      let _this = this;
      let socketClient = this.$store.getters.getSocketClient;
      let stompClient = this.$store.getters.getStompClient;

      socketClient = new SockJS(baseUrl + "/server?houseId="+this.houseId+"&housePwd="+this.housePwd+"&connectType="+this.connectType); ////// new SockJS("https://www.alang.run" + "/wss?houseId="+this.houseId+"&housePwd="+this.housePwd+"&connectType="+this.connectType);/
      stompClient = Stomp.over(socketClient);

      if (isProduction) {
        stompClient.debug = () => {};
      }
      stompClient.connect(
        {},
        frame => {
          // console.log('连接到服务器成功！', frame);
          this.$store.commit("setSocketIsConnected", true);
          // pre onmessage
          let afterOnMessage = socketClient.onmessage;
          socketClient.onmessage = function(message) {
            _this.messageHandler(message);
            afterOnMessage(message);
          };

          // pre onclose
          let afterOnclose = socketClient.onclose;
          socketClient.onclose = function(e) {
            if (e.type === "close") {
              _this.$store.commit("setSocketIsConnected", false);
              _this.$store.commit("pushChatData", {
                type: "notice",
                content: "网络异常, 请尝试重新连接服务器!"
              });
              _this.$toast.error("网络异常, 请尝试重新连接服务器!");
              setTimeout(function(){
                if(!_this.$store.getters.getIsConnected){
                  _this.connect();
                }
              },444);
            }
            afterOnclose(e);
          };

          let userName = window.localStorage.getItem("USER_NAME");
          if (userName) {
            this.settingName(userName);
          }

          this.subscribe();
        },
        error => {
          // console.log('连接到服务器失败！', error);
        }
      );

      this.saveSocket(socketClient, stompClient);
    },
    close: function() {
      let socketClient = this.$store.getters.getSocketClient;
      let stompClient = this.$store.getters.getStompClient;

      stompClient.disconnect();
      socketClient.close();
      this.isPlay = false;
      this.playingId = "";
      this.getHomeHouses();

      this.saveSocket(socketClient, stompClient);
     
    },
    subscribe: function() {
      let stompClient = this.$store.getters.getStompClient;

      stompClient.subscribe("/topic/chat", response => {
        let body = JSON.parse(response.body);
        if (body.code == "20000") {
          this.$toast.message({message:"系统通知："+body.data,time: 30*1000, closeIcon: 'close',close:true});
        }
        //this.$store.commit("pushChatData", .data);
      });

      stompClient.subscribe("/topic/music/order", response => {
        // console.log('来自 /topic/music/order 频道的消息', response);
      });

      this.saveSocket(null, stompClient);
    },
    saveSocket: function(socketClient, stompClient) {
      if (socketClient !== null) {
        this.$store.commit("setSocketClient", socketClient);
      }
      if (stompClient !== null) {
        this.$store.commit("setStompClient", stompClient);
      }
    },
    messageFieldEnterHandler: function(e) {
      // 按下回车时判断是否发送消息，如果按下的是回车键并且没有按下 shift 键且没有正在输入中文，则发送消息
      if (e.keyCode === 13 && !e.shiftKey && !e.isComposing) {
        this.sendHandler();
      }
    },
    sendHandler: function() {
      // console.log('消息发送处理器');
      let stompClient = this.$store.getters.getStompClient;
      let chatMessage = this.$store.getters.getChatMessage;

      let instruction = sendUtils.parseInstruction(chatMessage);
      let content = "";

      switch (instruction) {
        case "点歌":
          content = sendUtils.parseContent(instruction, chatMessage);
          if (content === "") {
            // console.log('请输入音乐关键词', chatMessage)
          } else {
            stompClient.send(
              "/music/pick",
              {},
              JSON.stringify({
                name: content,
                source: this.sourceChat,
                sendTime: Date.now()
              })
            );
          }
          break;
        case "投票切歌":
          this.musicSkipVote();
          break;
        case "设置昵称":
          content = sendUtils.parseContent(instruction, chatMessage);
          if (content === "") {
            // console.log('请输入昵称', chatMessage)
          } else {
            this.settingName(content);
          }
          break;
        case "通知":
          content = sendUtils.parseContent(instruction, chatMessage);
          if (content === "") {
            // console.log('请输入公告', chatMessage)
          } else {
            stompClient.send("/chat/notice/" + content, {}, JSON.stringify({}));
          }
          break;
         case "公告":
          content = sendUtils.parseContent(instruction, chatMessage);       
          stompClient.send("/chat/announce", {}, JSON.stringify({ content: content}));
          break;
        case "root":
          content = sendUtils.parseContent(instruction, chatMessage);
          if (content === "") {
            // console.log('请输入 root 密码', chatMessage)
          } else {
            stompClient.send(
              "/auth/root",
              {},
              JSON.stringify({
                password: content,
                sendTime: Date.now()
              })
            );
          }
          break;
        case "admin":
          content = sendUtils.parseContent(instruction, chatMessage);
          if (content === "") {
            // console.log('请输入 admin 密码', chatMessage)
          } else {
            stompClient.send(
              "/auth/admin",
              {},
              JSON.stringify({
                password: content,
                sendTime: Date.now()
              })
            );
          }
          break;
        case "置顶音乐":
          content = sendUtils.parseContent(instruction, chatMessage);
          if (content === "") {
            // console.log('请输入要置顶的音乐 id', chatMessage)
          } else {
            stompClient.send(
              "/music/top",
              {},
              JSON.stringify({
                id: content,
                sendTime: Date.now()
              })
            );
          }
          break;
        case "删除音乐":
          content = sendUtils.parseContent(instruction, chatMessage);
          if (content === "") {
            // console.log('请输入要删除的音乐 id', chatMessage)
          } else {
            stompClient.send(
              "/music/delete",
              {},
              JSON.stringify({
                id: content,
                sendTime: Date.now()
              })
            );
          }
          break;
              case "设置默认列表":
          content = sendUtils.parseContent(instruction, chatMessage);
          if (content === "") {
            // console.log('请输入要删除的音乐 id', chatMessage)
          } else {
            stompClient.send(
              "/music/setDefaultPlaylist",
              {},
              JSON.stringify({
                id: content,
                source: this.sourceChat
              })
            );
          }
          break;
        case "清空列表":
          stompClient.send("/music/clear", {}, "");
          break;
        case "清空默认列表":
          stompClient.send("/music/clearDefaultPlayList", {}, "");
          break;
        case "音乐黑名单":
          stompClient.send("/music/blackmusic", {}, "");
          break;
         case "默认列表歌曲数":
          stompClient.send("/music/playlistSize", {}, "");
          break;
        case "用户黑名单":
          stompClient.send("/chat/blackuser", {}, "");
          break;
        case "调整音量":
          content = sendUtils.parseContent(instruction, chatMessage);
          if (content === "") {
            content = 0;
          }

          stompClient.send("/music/volumn/" + content, {}, "");

          break;
         case "倒计时退出":
          content = sendUtils.parseContent(instruction, chatMessage);
          if(!/^\d+$/.test(content)){
            this.$toast.message("请输入要在几分钟后退出");
          }else{
            this.setTimeToClose(content);
            this.$toast.message("设置成功，将在"+content+"分钟后退出");
          }
          break;
        case "取消退出":         
          this.setTimeToClose(0);
          this.$toast.message("取消成功");
          break;
        case "修改密码":
          content = sendUtils.parseContent(instruction, chatMessage);

          stompClient.send("/auth/adminpwd/" + content, {}, "");

          break;
        case "修改root密码":
          content = sendUtils.parseContent(instruction, chatMessage);

          stompClient.send("/auth/rootpwd/" + content, {}, "");

          break;
        case "投票切歌率":
          content = sendUtils.parseContent(instruction, chatMessage);

          stompClient.send("/music/vote/" + content, {}, "");

          break;
        case "点赞模式":
          stompClient.send("/music/goodmodel/true", {}, "");
          break;
        case "退出点赞模式":
          stompClient.send("/music/goodmodel/false", {}, "");
          break;
        case "单曲循环":
          stompClient.send("/music/musiccirclemodel/true", {}, "");
        break;
          case "列表循环":
          stompClient.send("/music/musiclistmodel/true", {}, "");
        break;
        case "退出单曲循环":
          stompClient.send("/music/musiccirclemodel/false", {}, "");
          break;
        case "退出列表循环":
          stompClient.send("/music/musiclistmodel/false", {}, "");
          break;
        case "随机模式":
          stompClient.send("/music/randommodel/true", {}, "");
          break;
        case "退出随机模式":
          stompClient.send("/music/randommodel/false", {}, "");
          break;
           case "留存房间":
          stompClient.send("/house/retain/true", {}, "");
          break;
        case "不留存房间":
          stompClient.send("/house/retain/false", {}, "");
          break;
        case "禁止点歌":
          stompClient.send("/music/banchoose/true", {}, "");
          break;
        case "禁止切歌":
          stompClient.send("/music/banswitch/true", {}, "");
          break;
        case "启用切歌":
          stompClient.send("/music/banswitch/false", {}, "");
          break;
        case "启用点歌":
          stompClient.send("/music/banchoose/false", {}, "");
          break;
        case "拉黑用户":
          content = sendUtils.parseContent(instruction, chatMessage);
          if (content === "") {
            // console.log('请输入要拉黑的用户 session', chatMessage)
          } else {
            stompClient.send(
              "/chat/black",
              {},
              JSON.stringify({
                sessionId: content,
                sendTime: Date.now()
              })
            );
          }
          break;
        case "漂白用户":
          content = sendUtils.parseContent(instruction, chatMessage);
          if (content === "") {
            // console.log('请输入要漂白的用户 session', chatMessage)
          } else {
            stompClient.send(
              "/chat/unblack",
              {},
              JSON.stringify({
                sessionId: content,
                sendTime: Date.now()
              })
            );
          }
          break;
         case "设置点歌人":
          content = sendUtils.parseContent(instruction, chatMessage);
          if (content === "") {
            // console.log('请输入要拉黑的用户 session', chatMessage)
          } else {
            stompClient.send(
              "/auth/setPicker/"+content,
              {},
              ""
            );
          }
          break;
         case "取消点歌人":
          content = sendUtils.parseContent(instruction, chatMessage);
          if (content === "") {
            // console.log('请输入要拉黑的用户 session', chatMessage)
          } else {
            stompClient.send(
              "/auth/setNoPicker/"+content,
              {},
              ""
            );
          }
          break;
           case "设置切歌人":
          content = sendUtils.parseContent(instruction, chatMessage);
          if (content === "") {
            // console.log('请输入要拉黑的用户 session', chatMessage)
          } else {
            stompClient.send(
              "/auth/setVoter/"+content,
              {},
              ""
            );
          }
          break;
         case "取消切歌人":
          content = sendUtils.parseContent(instruction, chatMessage);
          if (content === "") {
            // console.log('请输入要拉黑的用户 session', chatMessage)
          } else {
            stompClient.send(
              "/auth/setNoVoter/"+content,
              {},
              ""
            );
          }
          break;
        case "拉黑音乐":
          content = sendUtils.parseContent(instruction, chatMessage);
          if (content === "") {
            // console.log('请输入要拉黑的音乐 id', chatMessage);
          } else {
            stompClient.send(
              "/music/black",
              {},
              JSON.stringify({
                id: content,
                sendTime: Date.now()
              })
            );
          }
          break;
        case "漂白音乐":
          content = sendUtils.parseContent(instruction, chatMessage);
          if (content === "") {
          } else {
            // console.log('请输入要漂白的音乐 id', chatMessage);
            stompClient.send(
              "/music/unblack",
              {},
              JSON.stringify({
                id: content,
                sendTime: Date.now()
              })
            );
          }
          break;
        case "@管理员":
          content = sendUtils.parseContent(instruction, chatMessage);
          if (content === "") {
          } else {
            stompClient.send(
              "/mail/send",
              {},
              JSON.stringify({
                content: content,
                sendTime: Date.now()
              })
            );
          }
          break;
        default:
          if (
            chatMessage === null ||
            chatMessage === "" ||
            chatMessage.length === 0
          ) {
            // console.log('消息非法', chatMessage);
          } else {
            stompClient.send(
              "/chat",
              {},
              JSON.stringify({
                content: chatMessage,
                sendTime: Date.now()
              })
            );
          }
          break;
      }

      this.$store.commit("setChatMessage", "");
    },
    messageHandler: function(source) {
      // console.log('消息处理器收到消息', source);
      if (messageUtils.isKnowMessageType(source.data)) {
        let messageType = messageUtils.parseMessageType(source.data);
        let messageContent = messageUtils.parseMessageContent(source.data);
        switch (messageType) {
          case messageUtils.messageType.ONLINE:
            if (
              messageContent.data.count !== undefined &&
              typeof messageContent.data.count !== "undefined" &&
              messageContent.data.count !== null &&
              messageContent.data.count !== ""
            ) {
              this.$store.commit("setSocketOnline", messageContent.data.count);
            }
            break;
          case messageUtils.messageType.HOUSE_USER:
            let users = messageContent.data;
            for(let i = 0; i < users.length; i++){
               this.$store.commit("pushChatData", {
                content: (i+1)+"."+users[i].nickName+"["+users[i].sessionId+"]",
                type: "notice"
              });
            }
            break;
          case messageUtils.messageType.NOTICE:
            if (
              messageContent.message !== undefined &&
              typeof messageContent.message !== "undefined" &&
              messageContent.message !== null &&
              messageContent.message !== ""
            ) {
              this.$store.commit("pushChatData", {
                content: messageContent.message,
                type: "notice"
              });
              if(messageContent.message=="点歌成功")
                this.$toast.message({message:messageContent.message,time:1000});
              }else{
                this.$toast.message(messageContent.message);
              }
            break;
          case messageUtils.messageType.ANNOUNCEMENT:
            if(this.announceToast){
                  this.$toast.close(this.announceToast);
                }
            if(messageContent.data.content){
                this.announceToast = this.$toast.message({message:"公告："+messageContent.data.content,time: 60*1000, closeIcon: 'close',close:true});
            }
            break;

          case messageUtils.messageType.CHAT:
            // parse picture
            let imgList = [];
            let matchUrlList = messageContent.data.content.match(
              /[picture].*?:\/\/[^\s]*/gi
            );
            if (matchUrlList !== null) {
              for (let i = 0; i < matchUrlList.length; i++) {
                imgList.push(matchUrlList[i].replace("picture:", ""));
                messageContent.data.content = messageContent.data.content.replace(
                  matchUrlList[i],
                  ""
                );
              }
            }
            messageContent.data.images = imgList;
            this.$store.commit("pushChatData", messageContent.data);
            break;
          case messageUtils.messageType.GOODMODEL:
            var data = messageContent.data;
            if (data == "GOOD") {
              this.$store.commit("setSocketGood", true);
            } else {
              this.$store.commit("setSocketGood", false);
            }
            // this.$forceUpdate();

            break;
          case messageUtils.messageType.PICK:
            if (messageContent.message == "goodlist") {
              this.$store.commit("setSocketGood", true);
            }
           this.$store.commit("setPlayerPick", messageContent.data);
            if (messageContent.data.length > 1) {
              this.secondUrl = messageContent.data[1].url;
              //console.log("this.firstLoaded"+this.firstLoaded);
              if (this.firstLoaded == 1) {
                this.$store.commit("setMusic2", { url: this.secondUrl });
              }
            }
            for(var i = 0; i < messageContent.data.length; i++){
              let itemIndex = this.containsArray(this.pickHistoryList,messageContent.data[i]);
              if(itemIndex != -1){
                this.pickHistoryList.splice(itemIndex,1);
              }else{
                if(this.pickHistoryList.length > 1000){
                  this.pickHistoryList.pop();
                }  
              }
              this.pickHistoryList.unshift(messageContent.data[i]);
            }
            localStorage.setItem("pickHistory",JSON.stringify(this.pickHistoryList));
            break;
          case messageUtils.messageType.VOLUMN:
            //console.log(messageContent.data);
            music.volume = Number(messageContent.data) / 100;
            this.$store.commit("setPlayerVolume", messageContent.data);

            break;
          case messageUtils.messageType.MUSIC:
            this.lastLyric="";
            this.$store.commit("setPlayerLyric", "");
            this.firstLoaded = 0;
            if(kuwoHttps != "" && messageContent.data.url.indexOf("kuwo.cn") != -1 && messageContent.data.url.indexOf("-") == -1){
              let urls = messageContent.data.url.split(".sycdn.");
              let headUrls = urls[0].replace('http://','').split(".");
              let lastHeadUrl = headUrls[headUrls.length-1];
              messageContent.data.url = 'https://'+lastHeadUrl+"-sycdn."+urls[1]+"&timestamp="+Date.now();
              //messageContent.data.url = kuwoHttps +"?timestamp="+Date.now()+"&targetUrl="+messageContent.data.url;
            }else{
              if(messageContent.data.url.indexOf("?") != -1){
              messageContent.data.url = messageContent.data.url+"&timestamp="+Date.now();
              }else{
              messageContent.data.url = messageContent.data.url+"?timestamp="+Date.now();
              }
              if(messageContent.data.url.indexOf("/ymusic/") != -1){
                messageContent.data.url = messageContent.data.url.replace(/(m\d+?)(?!c)\.music\.126\.net/, '$1c.music.126.net');//wy403RedirectUrl+"?targetUrl="+messageContent.data.url;
              }
            }
            messageContent.data.url = messageContent.data.url.replace("http://","https://");
            // console.log("看这里");
            // console.log(messageContent.data.url);
            // console.log(kuwoHttps);
            // console.log(messageContent.data.url.indexOf("kuwo.cn") != -1);
            this.$store.commit("setPlayerMusic", messageContent.data);
            document.querySelector("#music").preload = "auto";
            if (
              messageContent.data.lyric === undefined ||
              typeof messageContent.data.lyric === "undefined" ||
              messageContent.data.lyric === null ||
              messageContent.data.lyric === ""
            ) {
              this.$store.commit("setPlayerLyrics", []);
            } else {
              this.lyrics = musicUtils.parseLyric(messageContent.data.lyric);
              this.$store.commit(
                "setPlayerLyrics",
                this.lyrics
              );
            }
            document.title = messageContent.data.name;
            break;
          case messageUtils.messageType.AUTH_ROOT:
            this.$store.commit("pushChatData", {
              content: messageContent.message,
              type: "notice"
            });
            if (Number(messageContent.code) === 20000) {
              this.$store.commit("setSocketRoot", true);
              // console.log('root success')
            } else {
              this.$store.commit("setSocketRoot", false);
            }
            break;
          case messageUtils.messageType.ENTER_HOUSE_START:
             if (Number(messageContent.code) === 20000) {
                this.$store.commit("setPlayerPick", new Array());
             }
            break;
           case messageUtils.messageType.ADD_HOUSE_START:
             if (Number(messageContent.code) === 20000) {
                this.$store.commit("setPlayerPick", new Array());
             }
            break;
          case messageUtils.messageType.ENTER_HOUSE:
            this.loading.close();
            if (Number(messageContent.code) === 20000) {
              this.houseId = this.houseIdNoAction;
              this.housePwd = this.housePwdNoAction;
              this.connectType = this.connectTypeNoAction;
              this.musichouse = this.houseForward;
              // console.log('root success')
              this.openHouse = false;
              document
                .querySelectorAll(".mu-tooltip")
                .forEach(el => (el.style.display = "none"));
              let userName = window.localStorage.getItem("USER_NAME");
              if (userName) {
                this.settingName(userName);
               }
            } else {
              this.$toast.message(messageContent.message);
              this.getHouses();
            }
            break;
          case messageUtils.messageType.ADD_HOUSE:
            this.loading.close();
            if (Number(messageContent.code) === 20000) {
              this.musichouse = this.house.name;
              this.houseId = messageContent.data;
              this.housePwd = this.house.password;
              this.connectType = "";
              // console.log('root success')
              this.openHouse = false;
              document
                .querySelectorAll(".mu-tooltip")
                .forEach(el => (el.style.display = "none"));
              let userName = window.localStorage.getItem("USER_NAME");
              if (userName) {
                this.settingName(userName);
               }
            } else {
              this.$toast.message(messageContent.message);
            }
            break;
          case messageUtils.messageType.AUTH_ADMIN:
            this.$store.commit("pushChatData", {
              content: messageContent.message,
              type: "notice"
            });
            if (Number(messageContent.code) === 20000) {
              this.$store.commit("setSocketAdmin", true);
              // console.log('admin success')
            } else {
              this.$store.commit("setSocketAdmin", false);
            }
            break;
          case messageUtils.messageType.SETTING_NAME:
            this.$store.commit("pushChatData", {
              content: messageContent.message,
              type: "notice"
            });
            this.$store.commit("setSocketUserName", messageContent.data.name);
            break;
          case messageUtils.messageType.SEARCH:
            this.$store.commit("setSearchCount", messageContent.data.totalSize);
            this.$store.commit("setSearchData", messageContent.data.data);
            break;
          case messageUtils.messageType.SEARCH_SONGLIST:
            this.$store.commit("setSearchCountGd", messageContent.data.totalSize);
            this.$store.commit("setSearchDataGd", messageContent.data.data);
            break;
           case messageUtils.messageType.SEARCH_USER:
            this.$store.commit("setSearchCountUser", messageContent.data.totalSize);
            this.$store.commit("setSearchDataUser", messageContent.data.data);
            break;
          case messageUtils.messageType.SEARCH_HOUSE:
            this.houses = this.sortByPopulation(messageContent.data);
            break;
          case messageUtils.messageType.SEARCH_PICTURE:
            this.$store.commit(
              "setSearchPictureCount",
              messageContent.data.totalSize
            );
            this.$store.commit(
              "setSearchPictureData",
              messageContent.data.data
            );
            break;
          default:
            // console.log('未知消息类型', messageType, source);
            break;
        }
      }
    },
    containsArray:function(array, item) {
      return array.findIndex(obj => obj.id === item.id);
    },
    updateChatMessage: function(value) {
      this.$store.commit("setChatMessage", value);
    },
    updateSearchKeyword: function(value) {
      this.$store.commit("setSearchKeyword", value);
    },
    updateSearchKeywordGd: function(value) {
      this.$store.commit("setSearchKeywordGd", value);
    },
     updateSearchKeywordUser: function(value) {
      this.$store.commit("setSearchKeywordUser", value);
    },
    settingName: function(name) {
      let stompClient = this.$store.getters.getStompClient;
      stompClient.send(
        "/setting/name",
        {},
        JSON.stringify({
          name: name,
          sendTime: Date.now()
        })
      );
    },
    search: function() {
      let stompClient = this.$store.getters.getStompClient;
      stompClient.send(
        "/music/search",
        {},
        JSON.stringify({
          name: this.$store.getters.getSearchKeyword.trim(),
          sendTime: Date.now(),
          source: this.source,
          pageIndex: this.current,
          pageSize: this.limit
        })
      );
    },
    searchGd: function() {
      let stompClient = this.$store.getters.getStompClient;
      stompClient.send(
        "/music/searchsonglist",
        {},
        JSON.stringify({
          name: (this.$store.getters.getSearchKeywordGd+"").trim(),
          sendTime: Date.now(),
          source: this.sourceGd,
          pageIndex: this.currentGd,
          pageSize: this.limit
        })
      );
    },
     searchUser: function() {
      let stompClient = this.$store.getters.getStompClient;
      stompClient.send(
        "/music/searchuser",
        {},
        JSON.stringify({
          nickname: this.$store.getters.getSearchKeywordUser.trim(),
          sendTime: Date.now(),
          source: this.sourceUser,
          pageIndex: this.currentUser,
          pageSize: this.limit
        })
      );
    },
    paginationChange: function(page) {
      this.current = page;
      this.search();
    },
     paginationChangeGd: function(page) {
      this.currentGd = page;
      this.searchGd();
    },
     paginationChangeUser: function(page) {
      this.currentUser = page;
      this.searchUser();
    },
    goodMusic: function(row) {
      let stompClient = this.$store.getters.getStompClient;
      stompClient.send("/music/good/" + row.id, {}, {});
      this.$toast.message(`[${row.id}]${row.name} - 已发送点赞请求`);
    },
    pickMusic: function(row,quality) {
      let stompClient = this.$store.getters.getStompClient;
      stompClient.send(
        "/music/pick",
        {},
        JSON.stringify({
          name: row.name,
          id: row.id,
          source: this.source,
          quality:quality?quality:'320k',
          sendTime: Date.now()
        })
      );      this.$toast.message(`[${row.id}]${row.name} - 已发送点歌请求`);
    },
     pickMusicNoToast: function(row,quality) {
      let stompClient = this.$store.getters.getStompClient;
      stompClient.send(
        "/music/pick",
        {},
        JSON.stringify({
          name: row.name,
          id: row.id,
          source: row.source,
          quality:quality?quality:'320k',
          sendTime: Date.now()
        })
      );
    },

    showPickButton(value) {
      if (Number(value.st) < 0) {
        // 没有资源
        return false;
      } else if (Number(value.fl) === 0) {
        // 可能需要付费
        return false;
      }
      return true;
    },
     songlistDetail(value) {
       this.openSearchGd = false;
       this.openSearch = true;
       this.$store.commit("setSearchKeyword",  "*"+value.id);
       this.source = this.sourceGd.startsWith("wy")?(this.sourceGd.endsWith("userdj")?"wydt":"wy"):"qq";
       this.current = 1;
       this.search();
    },
     songtableDetail(value) {
       this.openSearchUser = false;
       this.openSearchGd = true;
       this.$store.commit("setSearchKeywordGd",  value.userId);
       this.sourceGd = this.sourceUser+"_user";
       this.currentGd = 1;
       this.searchGd();
    },
     djtableDetail(value) {
       this.openSearchUser = false;
       this.openSearchGd = true;
       this.$store.commit("setSearchKeywordGd",  value.userId);
       this.sourceGd = this.sourceUser+"_userdj";
       this.currentGd = 1;
       this.searchGd();
    },
    musicSkipVote: function() {
      let stompClient = this.$store.getters.getStompClient;
      stompClient.send("/music/skip/vote", {}, {});
    },
    houseUser: function() {
      let stompClient = this.$store.getters.getStompClient;
      stompClient.send("/house/houseuser", {}, {});
    },
    musicTimeUpdate: function(e) {
      // progress
      this.currentTime = e.target.currentTime;
      let duration = e.target.duration;
      this.$store.commit("setPlayerProgress", (this.currentTime / duration) * 100);

      // show time
      let usedTimeHH_mm_ss = timeUtils.secondsToHH_mm_ss(this.currentTime);
      let durationHH_mm_ss = timeUtils.secondsToHH_mm_ss(duration);
      let time = usedTimeHH_mm_ss + " / " + durationHH_mm_ss;
      this.$store.commit("setPlayerTime", time);

      // lyric
      this.lyrics = this.$store.getters.getPlayerLyrics;
      if (this.lyrics.length === 0) {
        this.$store.commit("setPlayerLyric", "暂无歌词");
      } else {
        let number = Number(this.currentTime.toFixed());
        if (this.lyrics[number] !== undefined && this.lyrics[number] !== "" && this.lyrics[number] != this.currentLyric) {
          this.lastLyric = this.currentLyric;
          this.currentLyric = this.lyrics[number];
          this.$store.commit("setPlayerLyric", this.lyrics[number]);
        }
      }
    },
    getScreenWidth: function() {
      const _this = this;
      window.onresize = () => {
        return (() => {
          _this.screenWidth = document.documentElement.clientWidth;
        })();
      };
    },
    formatterTime: function(value) {
      return timeUtils.secondsToHH_mm_ss(value);
    },
     formatterFullTime: function(value) {
      return timeUtils.secondsToYYYY_HH_mm_ss(value);
    },
    nextSong: function(e) {
      this.firstLoaded = 1;
      this.$store.commit("setMusic2", { url: this.secondUrl });
      //document.querySelector('#music2').src = this.secondUrl;

      //console.log(this.secondUrl);
    },
    closeHouse() {
      this.openHouse = false;
    },
     closeGd() {
      this.openSearchGd = false;
    },
    closeUser() {
      this.openSearchUser = false;
    },
    openGd(){
      this.openSearch = false;
      this.openSearchGd = true;
    },
    openGdFromUser(){
      this.openSearchUser = false;
      this.openSearchGd = true;
    },
     openGq(){
      this.openSearchGd = false;
      this.openSearch = true;
    },
       openUser(){
      this.openSearchGd = false;
      this.openSearchUser = true;
    },
     closeGq() {
      this.openSearch = false;
    },
    createHouse() {
      this.loading = this.$loading({ overlayColor: "hsla(0, 0%, 100%, .5)" });
      let stompClient = this.$store.getters.getStompClient;
      stompClient.send(
        "/house/add",
        {},
        JSON.stringify({
          name: this.house.name,
          desc: this.house.desc,
          needPwd:this.house.needPwd,
          password:this.house.password,
          enableStatus:this.house.enableStatus,
          retainKey:this.house.retainKey
        })
      );
    },
     createHomeHouse() {
      this.loading = this.$loading({ overlayColor: "hsla(0, 0%, 100%, .5)" });
       this.$http.post("/house/add",{  name: this.homeHouse.name,
          desc: this.homeHouse.desc,
          needPwd:this.homeHouse.needPwd,
          password:this.homeHouse.password,
          enableStatus:this.homeHouse.enableStatus,
          retainKey:this.homeHouse.retainKey
          })
        .then(response => {
          this.loading.close();
          if(response.data.code=="20000"){
            this.houseId = response.data.data;
            this.housePwd = this.homeHouse.password;
            this.connectType = "";
            this.play();
            this.musichouse = this.homeHouse.name;
                document
                .querySelectorAll(".mu-tooltip")
                .forEach(el => (el.style.display = "none"));
          }else{
            this.$toast.message(response.data.message);
          }
         
        })
        .catch(error => {
            this.loading.close();
        })
    },
    enterHouse(id, name, needPwd) {
      if (needPwd) {
        this.$prompt("", "请输入房间密码", {
          validator(value) {
            return {
              valid: value != "",
              message: "密码不能为空"
            };
          }
        }).then(({ result, value }) => {
          if (result) {
            this.houseEnter(id, name, value);
          } else {
            //this.$toast.message('点击了取消');
          }
        });
      } else {
        this.houseEnter(id, name, "");
      }
    },
     enterHomeHouse(id, name, needPwd) {
      if (needPwd) {
        this.$prompt("", "请输入房间密码", {
          validator(value) {
            return {
              valid: value != "",
              message: "密码不能为空"
            };
          }
        }).then(({ result, value }) => {
          if (result) {
            this.homeHouseEnter(id, name, value);
          } else {
            //this.$toast.message('点击了取消');
          }
        });
      } else {
        this.homeHouseEnter(id, name, "");
      }
    },
    houseEnter(id, name, pwd) {
      this.loading = this.$loading({ overlayColor: "hsla(0, 0%, 100%, .5)" });
      this.houseIdNoAction = id;
      this.housePwdNoAction = pwd;
      this.connectTypeNoAction = "enter";
      this.houseForward = name;
      let stompClient = this.$store.getters.getStompClient;
      stompClient.send(
        "/house/enter",
        {},
        JSON.stringify({
          id: id,
          password: pwd
        })
      );
    },
     homeHouseEnter(id, name, pwd) {
      this.loading = this.$loading({ overlayColor: "hsla(0, 0%, 100%, .5)" });

      this.$http.post("/house/enter",{ "id": id,
          "password": pwd})
        .then(response => {
          this.loading.close();
          if(response.data.code=="20000"){
            this.houseId = id;
            this.housePwd = pwd;
            this.connectType = "enter";
            this.play();
            this.musichouse = name;
             document
                .querySelectorAll(".mu-tooltip")
                .forEach(el => (el.style.display = "none"));
          }else{
            this.$toast.message(response.data.message);
          }
         
        })
        .catch(error => {
            this.loading.close();
        })
  
    },
    getHouses() {
      let stompClient = this.$store.getters.getStompClient;
      stompClient.send("/house/search", {}, JSON.stringify({}));
    },
    swap(x,y,houseList){
      let temp = houseList[x];
      houseList[x] = houseList[y];
      houseList[y] = temp;
    },
    compareLessThan(x,y,houseList){
      if(houseList[x].needPwd ^ houseList[y].needPwd){
        if(houseList[x].needPwd){
            if(houseList[y].population > 0){
              return true;
            }else if(houseList[x].population > 0){
              return false;
            }else{
              return true;
            }
        }else{
             if(houseList[x].population > 0){
              return false;
            }else if(houseList[y].population > 0){
              return true;
            }else{
              return false;
            }
        }
      }else{
        return houseList[x].population<houseList[y].population;
      }
    },
    sortByPopulation(houseList){
      let size = houseList.length;
      for(let i = 0; i < size-1;i++){
        for(let j = 0; j < size-i-1;j++ ){
          if(this.compareLessThan(j,j+1,houseList)){
            this.swap(j,j+1,houseList);
          }
        }
      }
      return houseList;
    },
    getHomeHouses() {
       this.$http.post("/house/search",{})
        .then(response => {
          if(response.data.code=="20000"){
            if(response.data.data[0].announce && response.data.data[0].announce.content){
                    this.$toast.message({message:response.data.data[0].announce.content,time: 30*1000, closeIcon: 'close',close:true})
            }
            this.homeHouses = this.sortByPopulation(response.data.data);
          }
         
        })
        .catch(error => {
        })
     
    },
    clearScr() {
      document.getElementById("chat-container").innerHTML = "";
    },
    setCurrentTime() {
      this.playingId= this.$store.getters.getPlayerMusic.id + this.$store.getters.getPlayerMusic.source+this.$store.getters.getPlayerMusic.pushTime;
    
    },
    linkDownload (url) {

      window.open(url,'_blank') // 新窗口打开外链接

    },
    playMusic(){
        document.querySelector("#music").play();
    },
    createTouchstartEventAndDispatch (el) {
      try{
         let event = document.createEvent('Events');
         event.initEvent('touchstart', true, true);
         el.dispatchEvent(event);
      }catch(Exception){
      }
    },
    //生成二维码
    getQRcode() {
      this.homeDesc = "";
       this.$http.post("/house/get",{  id: this.houseId})
        .then(response => {
          if(response.data.code=="20000"){
            this.homeDesc = response.data.data.desc;
          }else{
            this.$toast.message(response.data.message);
          }
         
        })
        .catch(error => {
        })
      this.qrcodeVue.value = this.joinUrl();
    },
     getMiniCode() {
       this.$http.post("/house/getMiniCode",{  id: this.houseId})
        .then(response => {
          if(response.data.code=="20000"){
            this.miniQrcode = "data:image/jpeg;base64,"+response.data.data;
          }else{
            this.$toast.message(response.data.message);
          }
         
        })
        .catch(error => {
        })
      this.qrcodeVue.value = this.joinUrl();
    },
    joinUrl(){
      let queryString = "houseId="+this.houseId+"&housePwd="+this.housePwd;
      let index = location.href.replace("//","||").indexOf("/");
      return (location.href.substring(0,index+1))+"?"+encodeURIComponent(queryString);	// 二维码内容
    },
    reachHouse(){
      let housePwd = this.getUrlKey("housePwd");
      this.homeHouseEnter(this.houseReachId,this.houseReachName,housePwd);
    },
    getUrlKey(name){
    if(window.location.href.indexOf("?houseId") != -1){
       let reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)", "i"); 
       let r = decodeURIComponent(window.location.search).substr(1).match(reg);  //获取url中"?"符后的字符串并正则匹配
       let context = ""; 
       if (r != null) 
          context = r[2]; 
       reg = null; 
       r = null; 
       return context == null || context == "" || context == "undefined" ? "" : context; 
    }else{
      return null;
    }
    },
    setTimeToClose(minutes){
      if(this.closeClock){
        window.clearTimeout(this.closeClock);
      }
      if(minutes != 0){
          this.closeClock = window.setTimeout(this.close,minutes*60*1000);
      }
    },
    searchTop(){
       this.openSearch = true;
       this.$store.commit("setSearchKeyword",  "*热歌榜");
       this.source = "wy";
       this.current = 1;
       this.search();
    },
    collectMusic(row){
      row.pickTime = new Date().getTime();
      this.$set(this.favoriteMap, row.id, row)

      // this.favoriteMap[row.id]= row;
      localStorage.setItem("collectMusic",JSON.stringify(this.favoriteMap));
    },
    removeCollect(row){
      this.$delete(this.favoriteMap,row.id);
      // delete this.favoriteMap[row.id];
      localStorage.setItem("collectMusic",JSON.stringify(this.favoriteMap));
    },
    containCollect(id){
      let result = this.favoriteMap[id] != null;
      return result;
    },
     closeBottomSheet () {
      this.open = false;
    },
    openBotttomSheet () {
      this.open = true;
    },
    openBotttomPickHistorySheet () {
      this.openPickHistory = true;
    },
    closeBottomPickHistorySheet () {
      this.openPickHistory = false;
    },
    removeAllCollect(){
      localStorage.removeItem("collectMusic");
      this.favoriteMap = {};
      this.open = false;
    },
    removeAllPickHistory(){
      localStorage.removeItem("pickHistory");
      this.pickHistoryList=[];
      this.openPickHistory= false;
    },
    seeMyselfPickHistory() {
      if(!this.$store.getters.getSocketUserName){
        this.$toast.message('请先设置昵称，可查看右下角教程');
      }
      this.pickSearch = this.$store.getters.getSocketUserName;
    },
    seeAllPickHistory() {
      // 看所有历史记录
      this.pickSearch = null;
    },
   
    exportCollect(){
      const dataStr = JSON.stringify(this.favoriteMap, null, 2);
      const dataUri = 'data:application/json;charset=utf-8,' + encodeURIComponent(dataStr);
      const exportFileDefaultName = 'music_'+timeUtils.secondsToYYYY_MM_dd_HH_mm_ss()+'.json';
      const linkElement = document.createElement('a');
      linkElement.setAttribute('href', dataUri);
      linkElement.setAttribute('download', exportFileDefaultName);
      linkElement.click();
    },
    importCollect(){
       this.$refs.fileInput.click();
    },
    handleFileInputChange(event) {
      const files = event.target.files;
      if (files.length === 0) return;
      const file = files[0];
      const reader = new FileReader();
      reader.onload = () => {
        try {
          const data = JSON.parse(reader.result);
          for(let key in data){
            this.$set(this.favoriteMap, key, data[key])
          }
          localStorage.setItem("collectMusic",JSON.stringify(this.favoriteMap));
          // 处理导入的JSON数据
        } catch (error) {
          console.error('JSON文件格式错误：', error);
        }
      };
      reader.readAsText(file);
    },
    playAll(){
      let i = 1;
      let _this = this;
      for(let key in this.favoriteMap){
        _this.pickMusicNoToast(_this.favoriteMap[key]);
      }
      this.open = false;
    },
    playCurrentPage(){
      for(let i = 0; i < this.searchData.length; i++){
        this.searchData[i].source = this.source;
        this.pickMusicNoToast(this.searchData[i]);
      }
    }
  },
  watch: {
    sourceGd:function(newValue,oldValue){
      this.currentGd=1;
      if(newValue=="qq_user"){
          this.placeHolderGd= "qq用户id即qq号";
      }else if(newValue =="wy_user"){
          this.placeHolderGd="用户id不知，点右上用户";
      }else{
          let placeholders = ["搜索[民谣]来听下吧","试下为空搜索(*^__^*)","请输入关键字，如'摇滚'"];
          this.placeHolderGd= placeholders[Math.floor(Math.random()*3)];
      }
    },
    source:function(newValue,oldValue){
         this.current=1;
         let placeholders = ["请输入关键字搜索,如'遇见'","歌单id搜索:'*歌单id'","不知听啥，点右上歌单..."];
         if(newValue == 'lz' || newValue == 'ai'){
          this.placeHolderGq= "试下为空搜索(*^__^*)"
         }else{
          this.placeHolderGq= placeholders[Math.floor(Math.random()*3)];
         }
    
    },
    playingId:function(newValue,oldValue){
      let _this = this;
      // console.log("新等于旧");
      if(newValue !="" && newValue != oldValue){
        this.albumRotate = false;
        document.querySelector("#music").volume =
        Number(this.$store.getters.getPlayerVolume) / 100;
        // console.log("新不等于旧");
      setTimeout(function() {
        _this.albumRotate = true;
        let pushTime = _this.$store.getters.getPlayerMusic.pushTime;
        if (pushTime) {
          // console.log("currenttime:",document.querySelector("#music").currentTime);
          if((Date.now() - pushTime) / 1000 - document.querySelector("#music").currentTime > 1){
             document.querySelector("#music").currentTime =
            (Date.now() - pushTime) / 1000;
          }
             _this.createTouchstartEventAndDispatch(document);
         
        }
      }, 1000);
      }
    },
    openHouse: function(newOpenHouse, oldOpenHouse) {
      if (newOpenHouse) {
        this.getHouses();
      }
    },
     openShare: function(newOpenHouse, oldOpenHouse) {
      if (newOpenHouse) {
        this.getQRcode();
        this.getMiniCode();
      }
    },
    "$store.state.player.music": function(newValue, oldValue) {
      // var audio = document.getElementById("music");
      // 解决部分移动端不能自动播放
      document.addEventListener("touchstart", function() {
        //_this.$toast.message("调用touchstart");
        document.getElementById("music").play();
        // audio.play();
      });

        if (window.WeixinJSBridge) {
            WeixinJSBridge.invoke('getNetworkType', {}, function (e) {
                document.getElementById("music").play();
            }, false);
        } else {
            document.addEventListener("WeixinJSBridgeReady", function () {
                WeixinJSBridge.invoke('getNetworkType', {}, function (e) {
                    document.getElementById("music").play();
                });
            }, false);
        }

      // wx.ready(function() {
      //     _this.$toast.message("调用weixin");
      //     document.querySelector("#music").play();
      // });
      // this.createTouchstartEventAndDispatch(document);
      // wx.config({
      //           debug:false,
      //           appId:"",
      //           timestamp:1,
      //           nonceStr:"",
      //           signature:"",
      //           jsApiList:[]
      //       });
      //       wx.ready(function(){
      //           var autoplayVideo=document.getElementById("music");
      //           autoplayVideo.play()
      //       })
      // this.albumRotate = false;
             // 解决部分移动端不能自动播放
      // document.addEventListener("touchstart", function() {
      //   document.querySelector("#music").play();
      // });
      // document.addEventListener("touchend", function() {
      //   let audio =  document.querySelector("#music");
      //   if(audio.paused){
      //     audio.play();
      //   }
      // });
      //  document.addEventListener("touchcancel", function() {
      //   let audio =  document.querySelector("#music");
      //   if(audio.paused){
      //     audio.play();
      //   }
      // });
      //  document.addEventListener("click", function() {
      //   let audio =  document.querySelector("#music");
      //   if(audio.paused){
      //     audio.play();
      //   }
      // });
      // setTimeout(function() {
      //   _this.albumRotate = true;
      //   let pushTime = _this.$store.getters.getPlayerMusic.pushTime;
      //   if (pushTime) {
      //     document.querySelector("#music").currentTime =
      //       (Date.now() - pushTime) / 1000;
      //   }
      // }, 1000);
    },
    "$store.state.chat.data": function(newValue, oldValue) {
      setTimeout(function() {
        let chat = document.querySelector("#chat-container");
        chat.scrollTop = chat.scrollHeight;
      }, 100);
    },
    screenWidth(val) {
      //监控浏览器高度变化
      if (!this.timer) {
        this.screenWidth = val;
        this.timer = true;
        let _this = this;
        setTimeout(function() {
          _this.timer = false;
        }, 400);
      }
      if (val <= 400) {
        this.albumRotateStyle =
          "border:60px solid rgb(12, 12, 12); padding: 8px;";
        this.pageCount = 3;
      }
      if (val > 400 && val <= 766) {
        this.albumRotateSize = 450;
        this.albumRotateStyle =
          "border:70px solid rgb(12, 12, 12); padding: 8px;";
        this.pageCount = 5;
      }
      if (val > 766 && val < 1000) {
        this.albumRotateSize = 160;
        this.albumRotateStyle =
          "border:32px solid rgb(12, 12, 12); padding: 4px;";
        this.pageCount = 9;

      }
      if (val >= 1000) {
        this.albumRotateSize = 200;
        this.albumRotateStyle =
          "border:40px solid rgb(12, 12, 12); padding: 4px;";
          this.pageCount = 11;

      }
      // console.log(this.pageCount+"dd");
    }
  },
  mounted() {
    setTimeout(balloons(),2000);
    this.getScreenWidth();
     this.$nextTick(function () {
      this.$http.defaults.baseURL = baseUrl;
      this.getHomeHouses();
      try{
          let houseId = this.getUrlKey("houseId");
          if(houseId){
            this.openShareReach = true;
            this.houseReachId=houseId;
            this.$http.post("/house/get",{  id: houseId})
              .then(response => {
              if(response.data.code=="20000"){
                    this.houseReachName = response.data.data.name;
              }else{
                  this.$toast.message(response.data.message);
              }
         
          })
            .catch(error => {
            })
         }
      }catch(Exception){

      }
   


    // Code that will run only after the
    // entire view has been rendered
    })
    wx.config({
            // 配置信息, 即使不正确也能使用 wx.ready
            debug: false,
            appId: '',
            timestamp: 1,
            nonceStr: '',
            signature: '',
            jsApiList: []
        });
    let collect =localStorage.getItem("collectMusic");
    if(collect && collect != undefined){
      this.favoriteMap = JSON.parse(collect);
      // console.log("收",this.favoriteMap);
    }
    let pickHistory =localStorage.getItem("pickHistory");
    if(pickHistory && pickHistory != undefined){
      this.pickHistoryList = JSON.parse(pickHistory);
    }
        // localStorage.removeItem("collectMusic");
  },
  created() {
    // let val = this.albumRotateSize;
    let val = document.documentElement.clientWidth;
    // console.log(val);

    if (val <= 400) {
      this.pageCount =3;
    }
    if (val > 400 && val <= 700) {
      this.albumRotateSize = val - 60;
      this.albumRotateStyle = `border:${Math.floor(val / 10) +
        10}px solid rgb(12, 12, 12);`;
      this.pageCount = 5;
    }
    if (val > 700 && val <= 766) {
      this.albumRotateSize = 450;
      this.albumRotateStyle = "border:70px solid rgb(12, 12, 12);";
            this.pageCount = 7;

    }
    if (val > 766 && val < 1000) {
      this.albumRotateSize = 160;
      this.albumRotateStyle = "border:32px solid rgb(12, 12, 12);";
            this.pageCount = 9;

    }
    if (val >= 1000) {
      this.albumRotateSize = 200;
      this.albumRotateStyle = "border:40px solid rgb(12, 12, 12);";
            this.pageCount = 11;

    }
  }
};
</script>

<style lang="scss" scoped>
.demo-container {
  .row {
    margin-bottom: 20px;

    &:last-child {
      margin-bottom: 0;
    }
  }
}

.album {
  width: 100%;
  display: inline-block;
  cursor: pointer;
  transition-duration: 0.2s;
  padding: 4px;
  border: 32px solid rgb(16, 16, 16);
  border-radius: 50%;
  background: linear-gradient(
    rgb(39, 39, 39),
    rgb(0, 0, 0),
    rgb(0, 0, 0),
    rgb(39, 39, 39)
  );
  /*box-shadow: 0 0 20px 2px #000;*/
}

.album-rotate {
  animation: rotate 20s linear infinite;
}

@keyframes rotate {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}
</style>
