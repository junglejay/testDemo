<template>
  <layout ref="layout" :scrollTop="scrollTop" pageType="scroll-view" @scroll2Top="scroll2Top">
    <view>
      <navbar :title="inspectTitle" :backData="this.list">
        <template #content>
          <!-- <p @click="handleAlarm"> 选择</p> -->
          <uni-icons type="search" color="#FFFFFF" size="30" class="searchBtn" @click="handleSearch()"></uni-icons>
          <uni-icons type="scan" color="#FFFFFF" size="30" @click="scanCode" v-if="mainDevOptions[mainDevIndex] && mainDevOptions[mainDevIndex].forcedNfcCheck != 1"></uni-icons>
        </template>
      </navbar>
      <template v-for="(mainDev, index) in list">
        <!-- :class="showBtn ? '' : 'keyIns'" -->
        <view class="inspectEdit"  v-if="mainDevIndex === index" :key="index">
          <view class="head">
            <view class="nodeName">
              <!-- {{mainDev.devName}} -->
              <uni-tooltip>
                <template>
                  <view class="uni-ellipsis">
                    {{ getFullName(mainDev.devName, 1) }}
                    <span>{{ getFullName(mainDev.devName, 2) }}</span>
                  </view>
                </template>
                <template #content>
                  <view style="white-space: break-spaces">
                    {{ getFullName(mainDev.devName, 1) }}
                    <span>{{ getFullName(mainDev.devName, 2) }}</span>
                  </view>
                </template>
              </uni-tooltip>
            </view>
            <view class="main inspectTitle">
              <view class="devType">{{ $$t('主设备') }}<image @click="handleOpenDevFilePopup" class="img" src="../../static/img/inspect/png.png"></image></view>
              <view class="flex_between_box">
                <view class="flex_left_box">
                  <view class="u-line-2">{{ mainDev.mainDevName }}</view>
                  <!-- 状态切换 -->
                  <view class="select" :class="[equipmentNowCode == 0 ? 'one' : equipmentNowCode == 4 ? 'two' : equipmentNowCode == 5 ? 'three' : equipmentNowCode == 6 ? 'four' : 'five']">
                    <span class="state"></span>
                    <span @click.stop="ulShow($event)">{{ equipmentNowState }}</span>
                  </view>
                </view>
                <view class="flex_right_box mainEquipmentNum" @click="mainEquipmentShow($event)">
                  <span :style="{ color: getCompleteNum == 0 ? null : !mainDevOptions[mainDevIndex].isChange ? '#2da641' : '#ed6a0c' }">{{ getCompleteNum }}</span
                  >/
                  <span>{{ allcheckPointNum }}</span>
                  <span>
                    <uni-icons type="right" size="18"></uni-icons>
                  </span>
                </view>
              </view>
            </view>
            <!-- 部分设备 -->
            <view class="inspectTitle part-dev">
              <view class="devType">{{ $$t('所属设备') }}</view>
              <view class="flex_between_box">
                <span class="partEquipment">{{ partEquipment }}</span>
                <view class="partEquipmentNum" @click="partEquipmentShow">
                  <uni-icons type="right" size="18"></uni-icons>
                </view>
              </view>
            </view>
          </view>
          <!-- 观察点渲染 -->
          <!-- <scroll-view :scroll-into-view="toView" :show-bar="false" :scroll-y="true" :scroll-with-animation="true" class="inspect-list"  v-if="equipmentNowState === '正常'">  :class="showBtn ? '' : 'keyUp'"-->
          <view class="inspect-list" v-if="equipmentNowCode == 0">
            <scroll-view :scroll-y="true" :scroll-top="scrollTop" @scroll="handleSVScroll" :scroll-into-view="toView" style="height: 100%">
              <view id="stateSetDetault" class="flex_right_box">
                <button v-if="!isView" style="margin: 20rpx 32rpx 0 0; padding: 0 44rpx" size="default" type="primary" @click="stateSetDetault">{{ $$t('一键默认') }}</button>
                <button v-if="mainDev.isEdit === false && !isView" style="margin: 20rpx 32rpx 0 0; padding: 0 44rpx" size="default" type="primary" @click="modify">{{ $$t('修改') }}</button>
              </view>
              <view class="inspectDetail" :id="'details' + secondaryDevIndex" v-for="(secondaryDev, secondaryDevIndex) in list[index].secondaryDevList" :key="secondaryDevIndex">
                <view class="title">
                  <span class="secondaryDevName">{{ secondaryDev.secondaryDevName }}</span>
                  <!-- <span v-if="mainDev.isEdit === false&&!isView" class="modify" @click="modify">{{ $$t('修改') }}</span> -->
                </view>
                <view v-for="(checkPoint, checkPointIndex) in secondaryDev.checkPointList" :key="checkPointIndex">
                  <view class="name">
                    <!-- <view class="left">
											<image :class="[checkPoint.pointType === '0' ? 'iconZero' :
									checkPoint.pointType === '1' ? 'iconOne' :
										checkPoint.pointType === '2' ? 'iconTwo' :
											checkPoint.pointType === '3' ? 'iconThree' : 'iconFour']" mode=""></image>
										</view> -->
                    <text :style="{ color: notFilledArr.includes(checkPoint.id) ? 'red' : null }">{{ checkPoint.pointName }}</text>
                    <text v-if="!isView" class="flex_center_box right" @click="setId(checkPoint)">•••</text>
                  </view>
                  <view class="content-warp">
                    <view class="content">
                      {{ $$t('测点标准') }}
                      <!-- <tips :msg="item.collectStatus"></tips> -->
                      <view id="problem">
                        <!-- v-show="checkPoint.pointType !== '0'" @click="popShow(checkPoint)" -->
                        <u-icon @click="toShowGuideMap(1, checkPoint)" style="margin-left: 20rpx" name="question-circle" color="gray" size="12"></u-icon>
                      </view>
                    </view>
                    <template>
                      <!-- 当pointType为0显示 -->
                      <view v-show="checkPoint.pointType === '0'">
                        <view class="singleChoice">
                          <u-checkbox-group v-show="mainDev.isEdit === true || (mainDev.isEdit === false && mainDev.isEditList && mainDev.isEditList.includes(checkPoint.id))" :value="[checkPoint.state]" placement="column" @change="runStateChange">
                            <u-checkbox class="content singleGroup chooseGreen" :customStyle="{ borderBottom: '1px solid #E4E7ED' }" shape="circle" icon-size="12" label-size="14" :label="$$t('运行正常')" :name="$$t('运行正常')" @change="radioChange($$t('运行正常'), checkPoint)"> </u-checkbox>
                            <u-checkbox v-if="checkPoint.pointObservation.length !== 1" class="content singleGroup chooseOrange" shape="circle" icon-size="12" label-size="14" :label="$$t('运行异常')" :name="$$t('运行异常')" @change="radioChange($$t('运行异常'), checkPoint)"> </u-checkbox>
                            <view v-if="checkPoint.pointObservation.length !== 1" class="chooses">
                              <view class="tip">{{ $$t('请选择异常情况(可多选)') }}</view>
                              <view class="checkbox">
                                <template v-for="(errorOption, errorOptionIndex) in checkPoint.pointObservation">
                                  <button
                                    v-if="errorOption.observationCode !== '0'"
                                    class="initialBtn flex_center_box choices"
                                    :class="{
                                      isChoices: checkPoint.checkedError.includes(errorOption.id)
                                    }"
                                    :disabled="errorOption.observationCode === '0'"
                                    :key="errorOptionIndex"
                                    @click="getChooseId(errorOption, checkPoint)"
                                  >
                                    {{ errorOption.observationName == '其他' ? $$t('其他') : errorOption.observationName }}
                                  </button>
                                </template>
                              </view>
                              <view class="other-option" v-if="checkPoint.showOther">
                                <view class="flex_between_box tip">
                                  <view>
                                    {{ $$t('其他异常情况') }}
                                  </view>
                                  <view
                                    class="alarm-lv"
                                    :style="{
                                      color: alarmColor(checkPoint.otherInfo.alarmLv)
                                    }"
                                    :id="'alarmSelect' + checkPointIndex"
                                    @click="showAlarmSelect(checkPointIndex, checkPoint.otherInfo.alarmLv, index, secondaryDevIndex, $event)"
                                  >
                                    {{ alarmLevelDic(checkPoint.otherInfo.alarmLv) }}
                                    <uni-icons type="bottom" color="#CECECE"></uni-icons>
                                  </view>
                                </view>
                                <u--textarea :placeholder="$$t('请输入异常情况')" @input="descriptionChange(checkPoint)" v-model="checkPoint.otherInfo.description"></u--textarea>
                              </view>
                            </view>
                          </u-checkbox-group>
                          <u-radio-group v-if="!(mainDev.isEdit === true || (mainDev.isEdit === false && mainDev.isEditList && mainDev.isEditList.includes(checkPoint.id)))" v-model="checkPoint.state">
                            <u-radio v-if="checkPoint.state" class="content singleGroup" :label="checkPoint.state" :name="checkPoint.state"> </u-radio>
                            <view v-else style="height: 24rpx"></view>
                          </u-radio-group>
                          <view v-if="checkPoint.state === $$t('运行异常') && !(mainDev.isEdit === true || (mainDev.isEdit === false && mainDev.isEditList && mainDev.isEditList.includes(checkPoint.id)))" class="chooses">
                            <view class="tip">{{ $$t('请选择异常情况(可多选)') }}</view>
                            <view class="checkbox">
                              <template v-for="(errorOption, errorOptionIndex) in checkPoint.pointObservation">
                                <button
                                  v-if="errorOption.observationCode !== '0'"
                                  class="initialBtn flex_center_box choices"
                                  :class="{
                                    isChoices: checkPoint.checkedError.includes(errorOption.id)
                                  }"
                                  :disabled="errorOption.observationCode === '0'"
                                  :key="errorOptionIndex"
                                >
                                  {{ errorOption.observationName == '其他' ? $$t('其他') : errorOption.observationName }}
                                </button>
                              </template>
                            </view>
                            <view class="other-option" v-if="checkPoint.showOther">
                              <view class="flex_between_box tip">
                                <view>
                                  {{ $$t('其他异常情况') }}
                                </view>
                                <view
                                  class="alarm-lv"
                                  :style="{
                                    color: alarmColor(checkPoint.otherInfo.alarmLv)
                                  }"
                                  :id="'alarmSelect' + checkPointIndex"
                                >
                                  {{ alarmLevelDic(checkPoint.otherInfo.alarmLv) }}
                                  <uni-icons type="bottom" color="#CECECE"></uni-icons>
                                </view>
                              </view>
                              <u--textarea :placeholder="$$t('请输入异常情况')" @input="descriptionChange(checkPoint)" disabled v-model="checkPoint.otherInfo.description"></u--textarea>
                            </view>
                          </view>
                        </view>
                      </view>
                      <!-- 当pointType 不为0时显示 -->
                      <view class="inputBox" v-show="checkPoint.pointType !== '0'">
                        <SimonInputNumber v-if="(mainDev.isEdit === true || (mainDev.isEdit === false && mainDev.isEditList && mainDev.isEditList.includes(checkPoint.id))) && !isView" :max="999999.99" :precision="2" v-model="checkPoint.pointObservation[0].value" @input="inputChange(checkPoint)" @onBlur="onBlur(checkPoint)" :placeholder="$$t('请输入内容')" :style="{ color: val2alarmColor(checkPoint) }"></SimonInputNumber>
                        <!-- <u-input
                          v-if="mainDev.isEdit === true"
                          v-model="checkPoint.pointObservation[0].value"
                          @change="inputChange(checkPoint)"
                          :placeholder="$$t('请输入内容')"
                          border="surround"
                          type="digit"
                          pattern="^\d{0,7}(\.\d{0,2})?$"
                          :maxlength="9"
                          clearable
                          :color="val2alarmColor(checkPoint)"
                        ></u-input> -->
                        <span v-else style="float: left" :style="{ color: val2alarmColor(checkPoint) }">{{ checkPoint.pointObservation[0].value }}</span>
                        <!-- 单位 -->
                        <span class="unit">{{ checkPoint.engineeringUnit }}</span>
                      </view>
                    </template>
                  </view>
                  <view v-if="getUploadProgress(checkPoint.fileGroupId, checkPoint.fileGroup) != 0" class="enclosure-warp">
                    <view class="flex_between_box enclosure-title">
                      {{ $$t('附件') }}
                      <view>
                        {{ $$t('已上传') }}：
                        <text>{{ getUploadProgress(checkPoint.fileGroupId, checkPoint.fileGroup) }}</text>
                      </view>
                    </view>
                    <view class="content">
                      <view class="flex_between_box" v-for="file in checkPoint.fileGroup" @click="previewFile(file, undefined, checkPoint.fileGroup)" :key="file.id">
                        <view class="flex_left_box left">
                          <view class="flex_center_box icon">
                            <image v-if="$IMAGETYPES.includes(file.annexFileType)" mode="heightFix" src="@/static/img/fileIcon-img.png"></image>
                            <image v-else-if="$VIDEOTYPES.includes(file.annexFileType)" mode="heightFix" src="@/static/img/fileIcon-video.png"></image>
                            <image v-else-if="$AUDIOTYPES.includes(file.annexFileType)" src="@/static/img/fileIcon-mp3.png" mode="heightFix" class="docTemp"></image>
                            <image v-else src="@/static/img/svg/doc-temp.svg" mode="heightFix" class="docTemp"></image>
                          </view>
                          <view class="flex_between_box info">
                            <view class="uni-ellipsis">
                              {{ file.annexName }}
                            </view>
                            <view>
                              {{ file.size }}
                              <text>{{ $$t('已上传') }}</text></view
                            >
                          </view>
                        </view>
                        <view v-if="!isView && mainDev.isEdit" class="flex_center_box right" @click.stop="delAnnex(file)">
                          <image src="@/static/img/close.png" mode="scaleToFill" />
                        </view>
                      </view>
                      <view class="flex_between_box" v-for="file in fileMsg.filter(v => v.fileGroupId == checkPoint.fileGroupId)" @click="previewFile(file, 'fileMsg')" :key="file.id">
                        <view class="flex_left_box left">
                          <view class="flex_center_box icon">
                            <u--image v-if="$IMAGETYPES.includes(item.annexFileType)" width="100%" height="100%" src="@/static/img/fileIcon-img.png" mode="heightFix"></u--image>

                            <!-- <image v-if="$IMAGETYPES.includes(file.annexFileType)" mode="heightFix" src="@/static/img/fileIcon-img.png"></image> -->
                            <image v-else-if="$VIDEOTYPES.includes(file.annexFileType)" mode="heightFix" src="@/static/img/fileIcon-video.png"></image>
                            <image v-else-if="$AUDIOTYPES.includes(file.annexFileType)" src="@/static/img/fileIcon-mp3.png" mode="heightFix" class="docTemp"></image>
                            <image v-else src="@/static/img/svg/doc-temp.svg" mode="heightFix" class="docTemp"></image>
                          </view>
                          <view class="flex_between_box info">
                            <view class="uni-ellipsis">
                              {{ file.annexName }}
                            </view>
                            <view>
                              <!-- {{file.size}}  -->
                              <text class="notUploaded">{{ $$t('未上传') }}</text>
                            </view>
                          </view>
                        </view>
                        <view v-if="!isView && mainDev.isEdit" class="flex_center_box right" @click.stop="delAnnex(file, 'fileMsg')">
                          <image src="@/static/img/close.png" mode="scaleToFill" />
                        </view>
                      </view>
                    </view>
                  </view>
                </view>
              </view>
              <view style="height: 132rpx"></view>
            </scroll-view>
          </view>

          <!-- </scroll-view> -->
        </view>
      </template>
      <template v-if="showSubmit">
        <view style="height: 132rpx"></view>
        <view v-if="list.length > 0&& showSubmit" class="upAndDownBtn" ref="upAndDownBtn">
          <!-- <button class="up" type="default" size="mini" @click="theLast">{{ $$t("上一个") }}</button>
          <button class="down" type="primary" size="mini" @click="handleDialog">{{ $$t("上传并下一个") }}</button> -->
          <button v-if="list.length > 1" class="flex_center_box step" type="default" size="mini" :disabled="mainDevIndex == 0" @click="theLast">
            <uni-icons type="back" size="20" :color="mainDevIndex != 0 ? '#3B71E8' : '#ccc'" />
          </button>
          <button v-if="list.length > 1" :disabled="isView" class="flex_center_box upload batchUpload" size="mini" @click="handleDialog('batchUpload')">
            <text style="line-height: 1.5">{{ $$t('批量提交') }}</text>
            <text style="margin-left: 12rpx">({{ devAllCompleteNum }}/{{ list.length }})</text>
          </button>
          <!-- :disabled="getDevCompleteNum(mainDevIndex).completeNum==0" -->
          <button class="flex_center_box upload" :class="{ singleDevice: list.length == 1 }" style="border-left: 0" :disabled="isView" type="primary" size="mini" @click="handleDialog('upload')">
            <text>{{ $$t('提交') }}</text>
          </button>
          <button v-if="list.length > 1" class="flex_center_box step" type="primary" size="mini" :disabled="mainDevIndex == list.length - 1" @click="theNext">
            <uni-icons type="forward" size="20" :color="mainDevIndex != list.length - 1 ? '#3B71E8' : '#ccc'" />
          </button>
        </view>
      </template>

      <view class="equipmentShow">
        <Picker :show="mainShow" :defaultIndex="[mainDevIndex]" :columns="[mainDevOptions]" :title="$$t('切换主设备')" keyName="label" @cancel="mainCancel" @confirm="mainConfirm"> </Picker>
        <Picker :show="partShow" :defaultIndex="[partDevIndex]" v-if="partShow" :columns="list[mainDevIndex] ? [list[mainDevIndex].secondaryDevOptions] : []" :title="$$t('切换所属设备')" keyName="label" @cancel="partCancel" @confirm="partConfirm"> </Picker>
      </view>
      <!-- 选择附件类型 -->
      <uni-popup ref="enclosureAction" type="bottom">
        <view class="enclosureAction">
          <view class="title">{{ planDetail.secondaryDevName }}</view>
          <view class="title_sub">{{ planDetail.pointName }}</view>
          <view class="flex_between_box fileList">
            <view class="flex_column_center fileList-item" @click="selectClick(1)">
              <image src="../../static/img/inspect/png.png"></image>
              {{ $$t('拍照') }}
            </view>
            <view class="flex_column_center fileList-item" @click="selectClick(2)">
              <image src="../../static/img/inspect/MP3.png"></image>
              {{ $$t('录音') }}
            </view>
            <view class="flex_column_center fileList-item" @click="selectClick(3)">
              <image src="../../static/img/inspect/MP4.png"></image>
              {{ $$t('拍视频') }}
            </view>
          </view>
          <view class="flex_between_box enclosure_remark" @click="selectClick(4)">
            <text>{{ $$t('添加备注') }}</text>
            <uni-icons type="compose" color="#303133" size="22"></uni-icons>
          </view>
          <view @click="$refs.enclosureAction.close()" class="flex_center_box btn">{{ $$t('取消') }}</view>
        </view>
      </uni-popup>
      <!-- 描述description -->
      <u-modal style="line-height: 100rpx" @confirm="confirm" :title="title" :show="standardShow">
        <view class="slot-content">
          <rich-text :nodes="content"></rich-text>
        </view>
      </u-modal>

      <!-- 让用户确认信息 -->
      <uni-popup class="overhaulPop confirmPopup" ref="confirmPopup" type="center">
        <view class="confirmPopup-content" v-if="list.length > 0">
          <view class="title">{{ $$t('确认提交') }}</view>
          <view class="title_sub">{{ $$t('请再次确认提交的数据是否正确') }}</view>
          <view class="scroll-view">
            <scroll-view :scroll-y="true">
              <view v-for="(secondaryDev, secondaryDevIndex) in completePoint" :key="secondaryDevIndex" class="secondaryDev">
                <view class="secondaryDevName">{{ secondaryDev.secondaryDevName }}</view>
                <view class="flex_between_box point" v-for="(checkPoint, checkPointIndex) in secondaryDev.checkPointList" :key="checkPointIndex">
                  <view class="pointName">
                    {{ checkPoint.pointName }}
                  </view>
                  <template v-if="secondaryDev.equipmentNowCode == 0">
                    <!-- 当pointType为0显示 -->
                    <template v-if="checkPoint.pointType == 0">
                      <view class="singleChoice">
                        <view
                          class="pointState"
                          :style="{
                            color: checkPoint.state === $$t('运行异常') ? '#ED6A0C' : '#2DA641'
                          }"
                        >
                          {{ checkPoint.state }}
                        </view>
                        <template v-if="checkPoint.state === $$t('运行异常')">
                          <view class="flex_right_box errorOption" style="flex-direction: column">
                            <view v-for="(errorOption, errorOptionIndex) in checkPoint.pointObservation" :key="errorOptionIndex" v-show="checkPoint.checkedError.includes(errorOption.id)">
                              {{ errorOption.observationName == '其他' ? $$t('其他') : errorOption.observationName }}
                              <text v-if="errorOption.observationName == '其他' && checkPoint.showOther && checkPoint.otherInfo">：</text>
                              <template v-if="errorOption.observationName == '其他'">
                                <text
                                  v-if="checkPoint.showOther && checkPoint.otherInfo"
                                  class="alarm-lv"
                                  :style="{
                                    color: alarmColor(checkPoint.otherInfo.alarmLv)
                                  }"
                                  :id="'alarmSelect' + checkPointIndex"
                                >
                                  {{ alarmLevelDic(checkPoint.otherInfo.alarmLv) }}
                                </text>
                                <!-- <text class="other-option" v-if="checkPoint.showOther && checkPoint.otherInfo.description">
                                  （{{ checkPoint.otherInfo.description }}）
                                </text> -->
                              </template>
                            </view>
                            <template v-if="checkPoint.state === $t('运行异常')">
                              <view class="flex_right_box errorOption" style="flex-direction: column">
                                <view v-for="(errorOption, errorOptionIndex) in checkPoint.pointObservation" :key="errorOptionIndex" v-show="checkPoint.checkedError.includes(errorOption.id)">
                                  {{ errorOption.observationName == '其他' ? $t('其他') : errorOption.observationName }}
                                  <text v-if="errorOption.observationName == '其他' && checkPoint.showOther && checkPoint.otherInfo">：</text>
                                  <template v-if="errorOption.observationName == '其他'">
                                    <text
                                      v-if="checkPoint.showOther && checkPoint.otherInfo"
                                      class="alarm-lv"
                                      :style="{
                                        color: alarmColor(checkPoint.otherInfo.alarmLv)
                                      }"
                                      :id="'alarmSelect' + checkPointIndex"
                                    >
                                      {{ alarmLevelDic(checkPoint.otherInfo.alarmLv) }}
                                    </text>
                                    <!-- <text class="other-option" v-if="checkPoint.showOther && checkPoint.otherInfo.description">
                                      （{{ checkPoint.otherInfo.description }}）
                                    </text> -->
                                  </template>
                                </view>
                              </view>
                            </template>
                          </view>
                        </template>
                        <!-- 当pointType 不为0时显示 -->
                        <view class="inputBox" v-show="checkPoint.pointType !== '0'">
                          <span v-if="checkPoint.pointObservation[0].value" style="color: #333">{{ checkPoint.pointObservation[0].value }}</span>
                          <!-- <span v-else>-</span> -->
                          <!-- 单位 -->
                          <span class="unit">{{ checkPoint.engineeringUnit }}</span>
                        </view>
                      </view>
                    </template>
                  </template>
                  <template v-else>
                      <!-- 免检状态 -->
                      <view class="select" :class="[secondaryDev.equipmentNowCode == 0 ? 'one' : secondaryDev.equipmentNowCode == 4 ? 'two' : secondaryDev.equipmentNowCode == 5 ? 'three' : secondaryDev.equipmentNowCode == 6 ? 'four' : 'five']">
                        {{ secondaryDev.equipmentNowCode | getStatusName(equipmentState) }}
                      </view>
                  </template>
                </view>
              </view>
            </scroll-view>
          </view>
          <view class="flex_between_box btns">
            <button class="flex_center_box cancel" @click="$refs.confirmPopup.close()">
              {{ $$t('取消') }}
            </button>
            <button class="flex_center_box confirm" @click="handleDialog(type, true, true)">
              {{ $$t('确认无误') }}
            </button>
          </view>
        </view>
      </uni-popup>
      <!-- 确认提交，并成功 -->
      <uni-popup class="overhaulPop successPopup" ref="successPopup" type="center" :animation="false">
        <uni-popup-dialog :confirmText="$$t('确认')" :cancelText="$$t('取消')" @confirm="handleBack">
          <div>
            <image class="correct_img" src="../../static/img/correct.png" />
          </div>
          <div class="correct_title">{{ $$t('提交成功！') }}</div>
        </uni-popup-dialog>
      </uni-popup>
      <ul v-show="equipmentStateShow" :style="{ top: alarmTop + 'px', left: alarmLeft + 'px' }">
        <li v-for="(item, index) in equipmentState" :key="index" @click="stateChange(item)">
          <span>{{ item.state }}</span>
          <uni-icons type="checkmarkempty" v-show="equipmentNowCode == item.dicValue" size="20" color="#3B71E8"> </uni-icons>
        </li>
      </ul>
      <view class="alarm-select" v-show="alarmLevelShow" :style="{ top: alarmTop + 'px' }">
        <view v-for="(item, index) in alarmLevelList" :key="index" @click="alarmLevelChange(item)" class="alarm-item">
          <view class="icon" :style="{ background: item.alarmColor }"></view>
          <span :style="{ color: item.alarmColor }">{{ item.alarmName }}</span>
          <uni-icons type="checkmarkempty" v-show="alarmSelectValue === item.id" size="20" color="#3B71E8"></uni-icons>
        </view>
      </view>
      <u-modal :show="showScanTip" :title="scanTipText" :showCancelButton="showCancelButton" :confirmText="scanConfirmText" :cancelText="scanCancelText" @confirm="scanConfirm" @cancel="scanCancel"></u-modal>
      <u-toast ref="uToast"></u-toast>
      <view class="popCloseBox" v-if="popupShow" @click="popClose"></view>
    </view>
    <filePreview ref="filePreview"></filePreview>
    <!-- 防作弊提示框 -->
    <u-modal :show="showNFCTip" :title="NFCTipText" :showCancelButton="true" :confirmText="$$t('拍照')" :cancelText="$$t('继续')"  @confirm="NFCConfirm('continue')" @cancel="NFCConfirm('cancel')"></u-modal>
    <!-- 点检标准 -->
    <uni-popup ref="standardPopup">
      <view class="standardPopup">
        <view class="standardPopup_title">
          <view @click="onClickItem(0)" :class="current == 0 ? 'active' : ''" v-if="checkPointImg.pointType !== '0'">点检标准</view>
          <view @click="onClickItem(1)" :class="current == 1 && checkPointImg.pointType !== '0' ? 'active' : ''">引导图</view>
        </view>
        <view v-if="current == 0" class="slot-content">
          <rich-text :nodes="content"></rich-text>
        </view>
        <view class="standardPopup_img" v-else>
          <swiper :indicator-dots="fileStandardList.length" indicator-color="#e5e5e5" indicator-active-color="#007aff" :autoplay="false" circular>
            <swiper-item v-if="fileStandardList.length == 0">
              <view class="no_data">
                <view class="no_data_img">
                  <image src="../../static/img/unStading.png" mode=""></image>
                </view>
                <view class="no_data_title">{{ $$t('暂无数据') }}</view>
              </view>
            </swiper-item>
            <template v-else>
              <swiper-item v-for="item in fileStandardList" :key="item.fileId">
                <image mode="scaleToFill" :src="item.fileSrc"></image>
              </swiper-item>
            </template>
          </swiper>
        </view>
        <view class="standardPopup_btn" @click="toShowGuideMap(0)">确认</view>
      </view>
    </uni-popup>
    <!-- 主设备附件预览 -->
    <uni-popup ref="devFilePopup">
      <view class="devFilePopup">
        <view class="devFilePopup_title">
          <view>{{this.list[this.mainDevIndex] ? this.list[this.mainDevIndex].mainDevName : ''}}</view>
        </view>
        <view class="devFilePopup_img" @click="handleDevFilePhoto" v-if="list[mainDevIndex].isEdit">
          <image class="img" src="../../static/img/inspect/png.png"></image>
          <view >拍照</view>
        </view>
        <view class="devFilePopup_preview" v-if="devFilePath !== ''">
          <image v-if="list[mainDevIndex].isEdit" class="close" src="@/static/img/close.png"   @click="handleDevFileImgClose"/>
          <image class="img" mode="heightFix" :src="devFilePath" @click="previewFile(list[mainDevIndex].fileGroup[0], undefined, list[mainDevIndex].fileGroup)"></image>
        </view>
        <view v-else class="noData">
            {{ $$t('暂无图片') }}
        </view>
        <view class="devFilePopup_btn" @click="handleDevFileClose">确认</view>
      </view>
    </uni-popup>
  </layout>
</template>

<script>
import navbar from '../common/navbar.vue'
import { inspectListforapp, inspectDetailforapp, alarmLevelList, inspectSendResult, uploadBase64, listbyfilegroupids, delAnnex, datapointGetvaluebytagname, listbyfilegroupid, editAnnexInfo } from '../../api/index.js'
import Picker from '../common/u-picker/u-picker.vue'
import tips from '../common/bubbleTips.vue'
import { callApi } from '../../utils/cordova'
import Snowflake from '../../utils/snowflake'
import { filterEmoji, hasEmoji } from '@/utils/index.js'
import moment from 'moment'
import scrollTop2PageMixin from '@/pages/common/layout/compoments/scrollTop/scrollTop2PageMixin.js'
import SimonInputNumber from '@/components/Simon-input-number/input-number.vue'
import { mapGetters } from 'vuex'
import filePreview from '@/components/filePreview/filePreview.vue'
import { encode } from 'js-base64'
let tempSnowflake = new Snowflake(1, 1, 0) //雪花算法生成文件name
export default {
  components: {
    navbar,
    Picker,
    tips,
    SimonInputNumber,
    filePreview
  },
  mixins: [scrollTop2PageMixin],
  data() {
    return {
      isCheating: false,
      isScanning: false,
      token: uni.getStorageSync('token'),
      popupShow: false,
      inspectTitle: '',
      id: 0,
      businessId: '',
      list: [],
      mainShow: false,
      partShow: false,
      mainComplete: 0,
      completeNum: 0, // 主设备下测点完成数量
      allcheckPointNum: 0, //主设备下所有测点数量
      partComplete: 0,
      mainDevOptions: [],
      partEquipment: {},
      partpmentAllNum: 0,
      test: [],
      equipmentStateShow: false,
      equipmentNowState: this.$$t('正常'),
      equipmentNowCode: 0,
      equipmentState: [], // 设备状态
      singleChoice: [
        {
          name: this.$$t('运行正常'),
          disabled: false,
          color: 'green'
        },
        {
          name: this.$$t('运行异常'),
          disabled: false,
          color: 'orange'
        }
      ],

      mainDevIndex: 0,
      partDevIndex: 0,
      // 测点标准显示
      standardShow: false,
      // 存放点检结果数据
      values: [],
      ids: [],
      title: '',
      content: '',
      // 判断当前主设备是否已经编辑过
      isEdit: [],
      toView: '',
      showScanTip: false,
      NFCTipText: '',
      scanTipText: this.$$t('请先扫描二维码解锁设备！'),
      scanConfirmText: this.$$t('立即扫码'),
      scanCancelText: this.$$t('稍后处理'),
      showCancelButton: true,
      showNFCTip: false,
      itemId: null,
      planDetail: {},
      timeId: -1,
      inputId: null,
      inputVal: '',
      timerId: null,
      info: {},
      alarmLevelShow: false,
      alarmLeft: 0,
      alarmTop: 0,
      alarmLevelList: [],
      alarmSelectValue: 0,
      index1: 0,
      index2: 0,
      index3: 0,
      businessType: 'eamCheckPointExe', //eamScOpratePlanTime|eamScRepairPlanTime|eamScMajorPlanTime
      checkType: '',
      runType: 'online', //normal 为电脑调试时的值, online为开发环境部署时的值, mobile 为手机调试的值
      fileMsg: [], //离线上传文件
      fileGroupId: '',
      notFilledArr: [], //未填写的测点的id
      option: {},
      isView: false, //知否只是查看详情
      saveType: 'upload', //点击的时批量提交按钮还是提交按钮
      showBtn: true, //是否显示提交按钮,键盘弹起时不显示
      defaultPhoneHeight: '', //屏幕默认高度
      nowPhoneHeight: '', //屏幕现在的高度
      fileStandardList: [],
      current: 0,
      checkPointImg: {},
      showSubmit: true,
      isNfcView:false, //防作弊不能编辑
      mainDevCode: '',//当前设备的code
      photoOrigin: '',
      devFilePath: '',
      source: ''
    }
  },
  computed: {
    ...mapGetters(['dictList']),
    // 获取主设备下拉
    getMainSelect() {
      let newArr = []
      let arr = this.list
      // debugger
      for (let i in arr) {
        let second = arr[i].secondaryDevList
        let obj = {}
        obj.lable = arr[i].mainDevName
        obj.mainDevId = arr[i].mainDevId
        obj.complete = 0
        obj.slash = '/'
        for (let j in second) {
          let check = second[j].checkPointList
          let len = []
          for (let r in check) {
            len.push(check[r])
          }
          obj.total = len.length
        }
        newArr.push(obj)
      }
      return newArr
      // this.mainDevOptions.push({
      // 	label:mainDev.mainDevName,
      // 	mainDevId:mainDev.mainDevId,
      // 	complete: 0,
      // 	slash: '/',
      // 	total: data.length
      // })
    },
    getCompleteNum() {
      return this.getDevCompleteNum(this.mainDevIndex).completeNum
      let arr = []
      let arrList = this.list[this.mainDevIndex].secondaryDevList
      for (let n in arrList) {
        let check = arrList[n].checkPointList
        for (let t in check) {
          // debugger
          if (check[t].pointObservation) {
            if (check[t].pointType === '0') {
              //选择运行正常和运行异常
              if (check[t].state === undefined) {
                arr.push(check[t].pointObservation[0])
              }
            } else {
              //填写观察量
              if (check[t].pointObservation[0].value === null || check[t].pointObservation[0].value === '') {
                arr.push(check[t].pointObservation[0])
              }
            }
          }
        }
      }
      if (this.equipmentNowState != this.$$t('正常')) {
        //如果是免检状态就当作全部完成
        arr = []
      }
      return this.allcheckPointNum - arr.length
    },
    getPartpmentAllNum() {
      let arr = []
      let arrList = this.list[this.mainDevIndex].secondaryDevList
      // debugger
      for (let n in arrList) {
        let check = arrList[n].checkPointList
        for (let t in check) {
          if (check[t].pointObservation) {
            arr.push(check[t].pointObservation[0].value)
          }
        }
      }
      return arr.length
    },
    devAllCompleteNum() {
      //批量上传按钮显示的设备的测点全部完成的数量
      let devcompleteNum = 0
      for (let i = 0; i < this.list.length; i++) {
        if (this.getDevCompleteNum(i).isComplete) {
          devcompleteNum++
        }
      }
      return devcompleteNum
    },
    isEn() {
      return uni.getStorageSync('language') === 'en'
    },
    isCn() {
      return uni.getStorageSync('language') === 'cn'
    },
    // 收集已填写的测点
    completePoint(){
      let arrList = []
      this.list.forEach(item => {
        let list = item.secondaryDevList
        item.secondaryDevList = list.map(val => {
          val.equipmentNowCode = item.exemptionStatus
          return val
        })
        let arr = []
        list.forEach(lone => {
          lone.checkPointList.forEach(ltwo => {
            if(ltwo.pointObservation){
              if (ltwo.pointType == '0'){
                //选择运行正常和运行异常
                if (ltwo.state !== undefined) {
                  arr.push(lone)
                }
              }else{
                if (ltwo.pointObservation[0].value !== null && ltwo.pointObservation[0].value !== '') {
                  arr.push(lone)
                }
              }
            }
          })
        })
        // 有填写测点，或免检
        if(arr.length != 0 || !item.status){
          arrList = arrList.concat(item.secondaryDevList)
        }
      })
      return arrList
    }
  },
  filters:{
    getStatusName(val,equipmentState){
      const equipmentNowState = equipmentState.find(item => item.dicValue === val).state
      return equipmentNowState
    }
  },
  async onShow() {
    //添加：NFC判断, 该功能只支持安卓
    var userAgent = navigator.userAgent || navigator.vendor || window.opera;
    console.info(userAgent.match(/Android/i))
    if (userAgent.match(/Android/i)) {
      this.NFCstart()
    } 
  },
  mounted() {
    //监听软键盘获取当前屏幕高度的事件
    this.defaultPhoneHeight = window.innerHeight
    window.onresize = () => {
      this.nowPhoneHeight = window.innerHeight
    }
  },
  watch: {
    list: {
      handler(nval, oval) {
        // 获取vuex中数组
        this.$store.commit('changeList', {
          id: this.itemId,
          list: nval
        })
        console.log('list----------------------',this.$store.state.list)
      },
      deep: true
    },
    mainDevOptions: {
      handler(nval, oval) {
        console.log(11111111111111, nval)
        this.$store.commit('changeMainPop', {
          id: this.itemId,
          list: nval
        })
      }
    },
    mainDevIndex() {
      this.datapointGetvaluebytagname() //获取数采值
    },
    //软键盘弹起事件
    nowPhoneHeight() {
      if (this.defaultPhoneHeight != this.nowPhoneHeight) {
        //手机键盘被唤起了。
        this.showBtn = false
      } else {
        //手机键盘被关闭了。
        this.showBtn = true
      }
    }
    // equipmentNowState: {
    // 	handler (nval, oval) {
    // 		// debugger
    // 		this.$store.commit('changeRunState', nval)
    // 	}
    // }
  },
  async onLoad(option) {
    /**
     * uniapp 应用启动onLaunch方法，改为同步，执行后再执行页面加载onLoad方法（https://blog.csdn.net/yfx000/article/details/108186719）
     */
    await this.$onLaunched
    this.option = option
    this.businessId = option.id //用来和上一页的列表关联、通讯的
    // 获取vuex数组
    console.log(this.$store.state.list)
    //监听软键盘获取当前屏幕高度的事件
    this.defaultPhoneHeight = window.innerHeight
    window.onresize = () => {
      this.nowPhoneHeight = window.innerHeight
    }
    this.checkNet()
      .then(res => {
        console.log(res)
        this.getState()

        this.getData(true, option) //* 在线
      })
      .catch(err => {
        this.getOutlineState()
        this.getData(false, option)
      })
  },
  onUnload() {
    callApi("NfcPlugin", "stop", [])
    // #ifdef H5
    // fix by mehaotian 处理 h5 滚动穿透的问题
    document.getElementsByTagName('body')[0].style.overflow = 'visible'
    // #endif
    const completeNum = this.mainDevOptions.reduce((a, b) => {
      return a + b.complete
    }, 0) //已完成的数量
    let obj = {}
    obj.id = this.businessId
    obj.list = this.list
    obj.completeNum = completeNum
    uni.$emit('inspectComplete', obj)
  },
  onPullDownRefresh() {
    this.startPullDownRefresh()
  },
  methods: {
    handleOpenDevFilePopup(){
      this.photoOrigin = 'dev'
      console.info(this.list, this.mainDevIndex, '0000000000000000')
      const devFileGroupId = this.list[this.mainDevIndex].devFileGroupId;
      this.fileGroupId = devFileGroupId;
      this.$refs.devFilePopup.open('center')
      this.listbyfilegroupids()
    },
    handleDevFilePhoto(){
      this.photograph();
    },
    handleDevFileClose(){
      this.photoOrigin = ''
      this.$refs.devFilePopup.close()
    },
    NFCstart(){
      cordova.exec((success)=> {
        //验证成功后
        if(Array.isArray(success)){
          //this.NFCServicesCallBack('GZ20240131000002 1001012001001')
          let arr = success.join('')
          console.log(success, arr);
          this.NFCServicesCallBack(success.join(''))
        }
      }, (error)=> {}, "NfcPlugin", "start", []);
    },
    NFCServicesCallBack(data) {
      if(this.mainDevOptions[this.mainDevIndex].forcedNfcCheck != '1' ) return;
      //this.readData = data
      let code = data.split(' ')[0]
      if(this.mainDevOptions[this.mainDevIndex].mainQrCode.includes(code)){
        this.isNfcView = false
        this.mainDevOptions[this.mainDevIndex].isNfcView = false;
        this.list[this.mainDevIndex].isNfcView = false;
        this.showNFCTip = false;
        this.$refs.uToast.show({
          type: 'success',
          message: this.$$t('解锁成功')
        })

      }else{
        this.showNFCTip = true;
        this.NFCTipText = this.$$t('请靠近正确的设备!')

      }
		},
    NFCConfirm(type){
      this.showNFCTip = false;
      if(type == 'continue'){
        this.photoOrigin = 'dev'
        const devFileGroupId = this.list[this.mainDevIndex].devFileGroupId;
        this.fileGroupId = devFileGroupId;
        this.handleDevFilePhoto();
      }
    },
    secondaryDev2abnormalPoint(secondaryDev) {
      //判断当前设备是否有异常的测点
      let abnormalPoint = []
      secondaryDev.checkPointList.forEach(checkPoint => {
        if ((checkPoint.pointType == 0 && checkPoint.state === this.$t('运行异常')) || (checkPoint.pointType !== '0' && this.val2alarmColor(checkPoint) !== '#303133')) {
          abnormalPoint.push(checkPoint)
        }
      })
      return abnormalPoint
    },
    onClickItem(e) {
      if (this.current !== e) {
        this.current = e
      }
    },
    datapointGetvaluebytagname() {
      //点检根据数采标签获取数采值
      let tagNameList = []
      let arrList = this.list[this.mainDevIndex].secondaryDevList
      for (let n in arrList) {
        let check = arrList[n].checkPointList
        for (let t in check) {
          if (check[t].tagName) {
            //有数采标签且未采集；3为采集完成；2为采集超时，但是有采集；
            tagNameList.push(check[t])
          }
        }
      }
      if (tagNameList.length == 0) {
        return false
      }
      datapointGetvaluebytagname({ tagNameList: tagNameList.map(v => v.tagName) }).then(res => {
        const tagName2ValueList = res.result || []
        tagName2ValueList.forEach(v => {
          const check = tagNameList.find(vv => vv.tagName == v.tagName)
          if (check && v.gatherValue) {
            this.$set(check.pointObservation[0], 'gatherValue', v.gatherValue)
            if (check.collectStatus != 3 && check.collectStatus != 2) {
              //3为采集完成；2为采集超时，但是有采集；
              this.$set(check.pointObservation[0], 'value', v.gatherValue)
            }
          }
        })
      })
    },
    startPullDownRefresh() {
      this.mainDevOptions = []
      this.checkNet()
        .then(res => {
          console.log(res)
          this.equipmentState = []
          this.getState()

          this.getData(true, this.option) //* 在线
        })
        .catch(err => {
          this.getOutlineState()
          this.getData(false, this.option)
        })
    },
    getDevCompleteNum(mainDevIndex) {
      //获取设备的测点完成数
      let allcheckPointNum = 0
      let arr = []
      let ids = []
      let list = [] //必填附件,但录入结果没有传附件
      let arrList = this.list[mainDevIndex].secondaryDevList
      for (let n in arrList) {
        let check = arrList[n].checkPointList
        for (let t in check) {
          allcheckPointNum++
          // debugger
          if (check[t].pointObservation) {
            if (check[t].pointType === '0') {
              //选择运行正常和运行异常
              if (check[t].state === undefined) {
                arr.push(check[t].pointObservation[0])
                ids.push(check[t].id)
              }
              if (check[t].state !== undefined && check[t].forcedAnnexUpload == '1' && (!check[t].fileGroup || !check[t].fileGroup.length)) {
                list.push(check[t])
              }
            } else {
              //填写观察量
              if (check[t].pointObservation[0].value === null || check[t].pointObservation[0].value === '') {
                arr.push(check[t].pointObservation[0])
                ids.push(check[t].id)
              }
              if (check[t].pointObservation[0].value !== null && check[t].pointObservation[0].value !== '' && check[t].forcedAnnexUpload == '1' && (!check[t].fileGroup || !check[t].fileGroup.length)) {
                list.push(check[t])
              }
            }
          }
        }
      }
      const equipmentNowState = this.equipmentState.find(item => item.dicValue === this.list[mainDevIndex].exemptionStatus).state
      const equipmentNowCode = this.equipmentState.find(item => item.dicValue === this.list[mainDevIndex].exemptionStatus).dicValue
      if (equipmentNowCode != 0) {
        //如果是免检状态就当作全部完成
        arr = []
        ids = []
        list = []
      }
      return {
        isComplete: arr.length == 0,
        completeNum: allcheckPointNum - arr.length,
        ids,
        isFile: list.length == 0,
        list,
      }
    },
    stateSetDetault() {
      //一键默认，，把当前主设备的未选择运行正常|运行异常的都设置为正常
      // 循环当前主设备，获取全部观测点数量
      let arrList = this.list[this.mainDevIndex].secondaryDevList
      for (let j = 0; j < arrList.length; j++) {
        for (let k = 0; k < arrList[j].checkPointList.length; k++) {
          console.info(checkPoint, '666666666666')
          const checkPoint = arrList[j].checkPointList[k]
          if (checkPoint.pointType === '0' && checkPoint.state === undefined) {
            checkPoint.showOther = false
            checkPoint.checkedError = []
            //修改测点时间，如果有时间就不修改
            checkPoint.measDate = moment().format('YYYY-MM-DD HH:mm:ss')
            let checkArr = checkPoint.pointObservation.filter(item => {
              return item.value === '1' && item.alarmLv !== '0'
            })
            if (checkArr.length) {
              this.$set(checkPoint, 'state', this.$$t('运行异常'))
            } else {
              this.$set(checkPoint, 'state', this.$$t('运行正常'))
            }
          }
        }
      }
      this.inputChange() //更新切换主设备弹出的“完成数”
    },
    // 预览文件
    /**
     *
     * @param {*} file
     * @param {*} isFileMsg 是否离线环境的未上传文件
     * @param {*} fileList预览 支持图片左右滑动列进行预览的表
     */
    previewFile(file, isFileMsg, fileList) {
      console.log('预览', file, isFileMsg, fileType)
      let fileType = null
      let annexFileType = file.annexFileType
      if (this.$IMAGETYPES.includes(annexFileType)) {
        fileType = 'img'
      } else if (this.$AUDIOTYPES.includes(annexFileType)) {
        fileType = 'amr'
      } else if (this.$VIDEOTYPES.includes(annexFileType)) {
        fileType = 'mp4'
      } else {
        fileType = 'file'
      }
      switch (fileType) {
        case 'img':
          if (isFileMsg == 'fileMsg') {
            // const previewPath = `data:image/*;base64,${file.base64}`
            this.$refs.filePreview.previewFile(
              file,
              this.fileMsg.filter(v => this.$IMAGETYPES.includes(v.annexFileType))
            )
          } else {
            console.log('测试是否代码有生效==============', fileList)
            this.$refs.filePreview.previewFile(
              file,
              fileList.filter(v => this.$IMAGETYPES.includes(v.annexFileType))
            )
          }
          break
        case 'mp4':
          if (isFileMsg == 'fileMsg') {
            const previewPath = `data:video/mp4;base64,${file.base64}`
            this.$refs.layout.previewFile(previewPath, [], true, fileType)
          } else {
            this.$refs.layout.previewFile(file)
          }
          break
        case 'amr':
          if (isFileMsg == 'fileMsg') {
            const previewPath = `data:audio/amr;base64,${file.base64}`
            this.$refs.layout.previewFile(previewPath, [], true, fileType)
          } else {
            this.$refs.layout.previewFile(file)
          }
          break
        default:
          this.$refs.layout.previewFile(file)
          // callApi("MCDocumentViewer", "previewWithFiles", [
          //   //pc端能传其他类型的文件
          //   {
          //     url: file.filePath,
          //     type: file.annexFileType,
          //     title: file.annexName,
          //   },
          // ])
          //   .then((res) => {})
          //   .catch((err) => {
          //     console.log(err, "预览失败");
          //   });
          break
      }
    },
    delAnnex(file, trueDel) {
      //删除附件
      uni.showModal({
        content: this.$$t('您确定要删除') + '?',
        showCancel: true,
        success: e => {
          if (e.confirm) {
            if (trueDel == 'fileMsg') {
              //未上传的文件
              this.fileMsg = this.fileMsg.filter(v => v.fileUrl != file.fileUrl)
              this.$refs.layout.showToast({
                title: this.$$t('删除成功'),
                duration: 2000
              })
            } else {
              delAnnex({ ids: file.id }).then(res => {
                this.listbyfilegroupids()
                // 延迟弹窗是因为在封装的请求函数里有人用了uni.hideLoading
                setTimeout(() => {
                  this.$refs.layout.showToast({
                    title: this.$$t('删除成功'),
                    duration: 2000
                  })
                }, 50)
              })
            }
          }
        }
      })
    },
    //判断上传进度
    getUploadProgress(fileGroupId, uploadList = []) {
      const unUploadList = this.fileMsg.filter(v => v.fileGroupId == fileGroupId) //未上传的文件（离线的时候才会有值）
      const total = uploadList.length + unUploadList.length
      if (!total) {
        return 0
      }
      return `${((uploadList.length / total) * 100).toFixed(0)}%`
    },
    getOutlineState() {
      callApi('JimDBhelp', 'readInformation', [
        {
          key: 'djOptions',
          DBname: 'eamMobile'
        }
      ]).then(res => {
        if (res.jsonString) {
          const optionsData = JSON.parse(res.jsonString)
          // this.list = inspectData[index];
          console.log(optionsData, '====读取离线缓存的数据,并解析了jsonString')

          for (let i = 0; i < optionsData.dicList.length; i++) {
            let obj = {}
            obj.state = optionsData.dicList[i].dicShowmsg
            obj.dicValue = optionsData.dicList[i].dicValue
            this.equipmentState.push(obj)
          }

          this.alarmLevelList = optionsData.colorList
        }
      })
    },
    getData(boo, option) {
      // debugger
      this.inspectTitle = uni.getStorageSync('inspectTitle')
      this.checkType = option.checkType
      const index = option.index
      // debugger
      if (option.id) {
        const id = option.id
        this.itemId = option.id
        if (boo) {
          this.getDetail(id)
        } else {
          //* 调用离线缓存中的列表数据
          //* 如果没网络,调取原生接口获取list
          callApi('JimDBhelp', 'readInformation', [
            {
              key: 'djList',
              DBname: 'eamMobile'
            }
          ])
            .then(res => {
              let platform = uni.getSystemInfoSync().platform
              if (platform == 'ios') {
                //* ios判断逻辑
                console.log('我是iOS,结果是->', res)
              } else if (platform == 'android') {
                //* 安卓判断逻辑
                console.log('我是安卓,结果是->', res, '当前数组下标是->', index)
              }
              //* 关键数据
              if (res.jsonString) {
                const inspectData = JSON.parse(res.jsonString)
                // this.list = inspectData[index];
                this.processingData(inspectData[index].detailMsgList)
                console.log(inspectData, '====读取离线缓存的数据,并解析了jsonString')
              }
            })
            .catch(err => {
              console.log(err, 'cordova请求失败=======')
            })
        }
        uni.setStorage({
          key: 'dataId',
          data: id,
          success: function () {
            console.log('success')
          }
        })
      }
    },
    popClose() {
      this.popupShow = false
      this.equipmentStateShow = false
      this.alarmLevelShow = false
      this.mainShow = false
      this.partShow = false
    },
    scroll(e) {
      console.log(e.detail.scrollHeight, '高度')
      // 1. 监听当前页面滚动的长度
      // 2. 找到各个子项的高度，并动态赋值到data，写法如下
      // this[itemsHeight] = '当前项的高度'
      // 3.
    },
    getChooses(id, data) {
      for (let i = 1; i <= data.length; i++) {
        if (id === data[i].id) {
          return data[i].observationName
        }
      }
    },
    // 获取字典
    getState() {
      let arr = this.dictList['EXEMPTION_STATUS'] || []
      for (let i = 0; i < arr.length; i++) {
        let obj = {}
        obj.state = arr[i].dicShowmsg
        obj.dicValue = arr[i].dicValue
        this.equipmentState.push(obj)
      }
      alarmLevelList({ pageNum: 1, pageSize: 20 }).then(res => {
        this.alarmLevelList = res.result.list
      })
    },
    async getDetail(planexeId) {
      const { result: inspectDetail } = await inspectDetailforapp({ id: planexeId }).catch(err=>{
        setTimeout(()=>{
          uni.showToast({
            title: err.returnMsg || this.$$t('数据不存在'),
            icon: 'none',
            duration: 4000
          });
        },1000)
      })
      this.mainDevCode = inspectDetail.mainDevCode
      // 禅道bug38785。是否过期展示判断
      this.endTime = new Date(inspectDetail.endTime)
      if(this.endTime.getTime() < Date.now() && inspectDetail.appShowFlag == '0'){
        // 过期
        this.showSubmit = false
      }else{
        this.showSubmit = true
      }
      if ([2, 4, '2', '4'].includes(inspectDetail.exeStatus) && inspectDetail.endTime < moment().format('YYYY-MM-DD HH:mm:ss')) {
        //已完成，且过期的任务不可以再编辑
        this.isView = true
      }
      // debugger
      inspectListforapp({ planexeId: planexeId, isEditPlan: true }).then(res => {
        // debugger
        this.processingData(res.result)
        console.log(res, 'ckck')
      })
    },
    // 页面数据
    async processingData(data) {
      uni.stopPullDownRefresh()
      console.log(data, this.$store.state.list, '-----------陈凯')
      //* 在这里调用离线缓存, 把list数据存入本地, 如果没网络就请求callApi; 有网络就不必

      // debugger
      this.partpmentAllNum = data[this.mainDevIndex].secondaryDevList.length
      this.partEquipment = data[this.mainDevIndex].secondaryDevList[0].secondaryDevName
      // debugger

      // 循环当前主设备，获取全部观测点数量
      let arrList = data[this.mainDevIndex].secondaryDevList
      // debugger
      let array = []
      for (let n in arrList) {
        let check = arrList[n].checkPointList
        for (let t in check) {
          if (check[t].pointObservation) {
            array.push(check[t].pointObservation)
          }
        }
      }
      this.allcheckPointNum = array.length
      //调用附件批量接口
      const devFileGroupIds =  data.map(item=>{
        return item.devFileGroupId
      })
      
      let devFileGroupIdsAwait = await listbyfilegroupids({ fileGroupIds: devFileGroupIds })
      const devFileGroupIdsRerulets = devFileGroupIdsAwait.result

      for (let i = 0; i < data.length; i++) {
        data[i].isEdit = true
        const mainDev = data[i]
        mainDev.status = true
        // 子设备列表选项
        // mainDev.secondaryDevOptions = mainDev.secondaryDevList.map(item=>{
        // 	return{
        // 		label:item.secondaryDevName,
        // 		mainDevId:item.mainDevId,
        // 		total: mainDev.secondaryDevList.length
        // 	}
        // })
        // 主设备列表选项

        let array = []
        let complete = 0
        // debugger
        for (let j = 0; j < mainDev.secondaryDevList.length; j++) {
          const secondaryDev = mainDev.secondaryDevList[j]

          for (let k = 0; k < secondaryDev.checkPointList.length; k++) {
            const checkPoint = secondaryDev.checkPointList[k]

            array.push(secondaryDev.checkPointList[k].pointObservation)
            // debugger
            if (checkPoint.collectStatus == 3 || checkPoint.collectStatus == 2) {
              //3为采集完成；2为采集超时，但是有采集；
              if (checkPoint.pointObservation[0].value !== null && checkPoint.pointObservation[0].value !== '') {
                complete++
              }
            }

            if (checkPoint.pointType !== '0') {
              checkPoint.inputValues = ''
            } else if (checkPoint.pointType === '0') {
              checkPoint.showOther = false
              checkPoint.checkedError = []
              let checkArr = checkPoint.pointObservation.filter(item => {
                return item.value === '1' && item.observationCode != '0'
              })
              if (checkPoint.collectStatus == 3 || checkPoint.collectStatus == 2) {
                //3为采集完成；2为采集超时，但是有采集；
                if (checkArr.length) {
                  checkPoint.state = this.$$t('运行异常')
                } else {
                  checkPoint.state = this.$$t('运行正常')
                }
              }
              checkPoint.checkedError = checkArr.map(item => item.id)
              console.log(checkArr, '选中的')
              //checkPoint.pointObservation[1] = checkPoint.pointObservation.splice(checkPoint.pointObservation.length - 1, 1, checkPoint.pointObservation[1])[0]
              let otherObservation = checkPoint.pointObservation[checkPoint.pointObservation.length - 1]

              if (checkPoint.checkedError.some(item => item === otherObservation.id)) {
                checkPoint.otherInfo = {
                  alarmLv: Number(otherObservation.alarmLv),
                  description: otherObservation.description
                }
                checkPoint.showOther = true
              }
            }
            mainDev.secondaryDevOptions = mainDev.secondaryDevList.map(item => {
              return {
                label: item.secondaryDevName,
                mainDevId: item.mainDevId,
                // total: secondaryDev.checkPointList.length
                isForcedCodeScan: item.forcedCodeScan === '1' ? true : false,
                isForcedNfcCheck: item.forcedNfcCheck === '1' ? true : false
              }
            })
          }
        }
        this.equipmentNowState = this.equipmentState.find(item => item.dicValue === data[0].exemptionStatus).state
        this.equipmentNowCode = this.equipmentState.find(item => item.dicValue === data[0].exemptionStatus).dicValue
        const equipmentNowState = this.equipmentState.find(item => item.dicValue === data[i].exemptionStatus).state
        const equipmentNowCode = this.equipmentState.find(item => item.dicValue === data[i].exemptionStatus).dicValue
        if (equipmentNowCode != 0) {
          //如果是免检状态就当作全部完成
          complete = array.length
        }
        //查找对应的主设备id
        const devFileGroupIdsRerulet = devFileGroupIdsRerulets.find(item=>item.fileGroupId == data[i].devFileGroupId)
        console.info(devFileGroupIdsRerulet,data[i].devFileGroupId, 'devFileGroupIdsRerulet4444444444444')
        //判断主设备是否需要防作弊
        const forcedNfcCheck = this.$store.state.list[this.itemId] ? this.$store.state.list[this.itemId].find(item=>item.mainDevId == mainDev.mainDevId).forcedNfcCheck : mainDev.forcedNfcCheck
        const isNfcView = this.$store.state.list[this.itemId] ? this.$store.state.list[this.itemId].find(item=>item.mainDevId == mainDev.mainDevId).isNfcView : mainDev.isNfcView
        console.info(this.$store.state.list[this.itemId], isNfcView, mainDev, this.mainDevIndex, 'forcedNfcCheck999999999999999999')
        // 判断主设备是否扫码
        this.mainDevOptions.push({
          icon: mainDev.forcedCodeScan === '1' ? 'lock' : undefined,
          isForcedCodeScan: mainDev.forcedCodeScan === '1' || this.isScanning ? true : false,
          isForcedNfcCheck: mainDev.forcedNfcCheck === '1' || this.isCheating ? true : false,
          forcedCodeScan: mainDev.forcedCodeScan, //把是否需要强制扫码记录到这里，就算扫过码下次切换回来也要重新扫码，所以就需要这个临时变量
          forcedNfcCheck: forcedNfcCheck, 
          originForcedNfcCheck: mainDev.forcedNfcCheck,
          isNfcView: (forcedNfcCheck == '1' && !devFileGroupIdsRerulet && isNfcView != false) ? true : false,//nfc 刷过一次不再刷
          label: mainDev.mainDevName,
          mainDevId: mainDev.mainDevId,
          mainDevCode: mainDev.mainDevCode,
          mainQrCode: mainDev.mainQrCode,
          status: mainDev.status,
          complete: complete, //已完成填写的数量
          uploadNum: complete, //已完成上传的数量（由于这是页面初始化才会调用的，所以已上传数和已完成数是一样的。）
          slash: '/',
          total: array.length,
          isChange: false //从数据库获取的最新数据，从未改变过
        })
      }
      // 已采集数据,保留一份原始数据,取消选中自动回填
      this.list = data.map((item, index) => {
        item.secondaryDevList = item.secondaryDevList.map(sec => {
          sec.checkPointList = sec.checkPointList.map(che => {
            if (che.pointType == '0') {
              che.stateUp = che.state
              if (che.state != this.$$t('运行正常')) {
                che.checkedErrorUp = che.checkedError
                che.showOtherUp = che.showOther
                che.otherInfoUp = che.otherInfo
              }
            } else {
              che.pointObservation[0].valueUp = che.pointObservation[0].value
            }
            return che
          })
          return sec
        })
        const forcedNfcCheck = this.$store.state.list[this.itemId] ? this.$store.state.list[this.itemId].find(itemSub=>itemSub.mainDevId == item.mainDevId).forcedNfcCheck : item.forcedNfcCheck
        const isNfcView = this.$store.state.list[this.itemId] ? this.$store.state.list[this.itemId].find(itemSub=>itemSub.mainDevId == item.mainDevId).isNfcView : item.isNfcView
        return {
          ...item,
          forcedNfcCheck,
          isNfcView: isNfcView != false ? true : false,
          status: item.exemptionStatus == '0' ? true : false
        }
      })
      this.listbyfilegroupids()
      console.log(this.list, '修改后的所有数据')
      // console.log(this.isEdit, '主设备数量与设备是否')
      // 执行上一次操作赋值
      /**赤山代码 -start*/
      let obj = this.$store.state.list
      for (let i in obj) {
        if (i === this.itemId) {
          obj[i].forEach((v, i) => {
            let secondaryDevList = v.secondaryDevList
            for (let n in secondaryDevList) {
              let checkPointList = secondaryDevList[n].checkPointList
              for (let t in checkPointList) {
                if (checkPointList[t].pointObservation) {
                  if (checkPointList[t].pointType === '0') {
                    //选择运行正常和运行异常
                    if (checkPointList[t].state !== undefined) {
                      this.setOldVal({ mainDevId: v.mainDevId, pointId: checkPointList[t].pointId, oldCheckPoint: checkPointList[t] })
                    }
                  } else {
                    //填写观察量
                    if (checkPointList[t].pointObservation[0].value !== null && checkPointList[t].pointObservation[0].value !== '') {
                      this.setOldVal({ mainDevId: v.mainDevId, pointId: checkPointList[t].pointId, oldCheckPoint: checkPointList[t] })
                    }
                  }
                }
              }
            }
          })
        }
      }
      /**赤山代码 -end*/
      // 如果当前设备下的测点没有全部完成，把当前设备下已经完成的测点设置为修改状态
      this.list[this.mainDevIndex].isEdit = this.getCompleteNum == 0
      this.list[this.mainDevIndex].isEditList = this.getDevCompleteNum(this.mainDevIndex).ids //把没有数据的记录起来
      if (this.list[this.mainDevIndex].forcedCodeScan === '1' && this.list[this.mainDevIndex].isEdit) {
        this.showScanTip = true
      }
      this.datapointGetvaluebytagname() //获取数采值
      //如果是从首页扫码或者防作弊进来，需要修改主设备排序顺序
      console.log('11111判断顺序啦',this.option.devId )
      if(this.option.devId && this.option.devId !== 'undefined'){
        const mainDevIndex = this.list.findIndex(v => v.mainDevId == this.option.devId)
        this.mainDevOptions[mainDevIndex].isNfcView = false;
        this.list[mainDevIndex].isNfcView = false;
        this.mainConfirm({ indexs: [mainDevIndex] })
        this.showNFCTip = false
      }else{
        //判断第一个设备是否需要进行防作弊
        console.log(this.list[0])
        if(this.list[0].forcedNfcCheck == 1 && this.list[0].status && this.mainDevOptions[0].uploadNum == 0){
          this.isNfcView = true
          this.dialogNfcTip()
        }
      }
      console.log('this.endTime', this.endTime)
      if (this.source === 'izjBacklog' && moment(moment()).isAfter(moment(this.endTime))) { // i紫金待办进入 && 过期
        this.showScanTip = false
        this.showNFCTip = false
        uni.showModal({
          content: this.$t('该点检任务已过期，无法进行填写。用户点击确认后跳转到设备系统【首页】'),
          showCancel: false,
          success: res => {
            if (res.confirm) {
              uni.reLaunch({
                url: "/pages/index/index",
              })
            }
          }
        })
      }
    },
    setOldVal({ mainDevId, pointId, oldCheckPoint }) {
      //由于后端没有返会分设备id，所以只能循环判断
      const mainDevIndex = this.list.findIndex(v => v.mainDevId == mainDevId) //找到主设备
      this.list[mainDevIndex].secondaryDevList.forEach(secondaryDev => {
        secondaryDev.checkPointList.forEach((checkPoint, checkPointI) => {
          //找到对应的测点
          if (checkPoint.pointId == pointId && checkPoint.collectStatus != 3 && checkPoint.collectStatus != 2) {
            //3为采集完成；2为采集超时，但是有采集；已采集过的就用线上的数据
            this.$set(secondaryDev.checkPointList, checkPointI, oldCheckPoint)
            this.$set(this.mainDevOptions[mainDevIndex], 'complete', ++this.mainDevOptions[mainDevIndex].complete) //有本地操作记录的话，要给完成数加上
            this.$set(this.mainDevOptions[mainDevIndex], 'isChange', true) //使用本地操作记录的，变为未上传数据
          }
        })
      })
    },
    mainEquipmentShow() {
      this.mainShow = true
      this.popupShow = true
    },
    partEquipmentShow() {
      this.partShow = true
      this.popupShow = true
    },
    mainCancel() {
      this.mainShow = false
      this.popupShow = false
    },
    // 主设备选择
    mainConfirm(val) {
      this.isNfcView = false
      this.showScanTip = false
      this.mainDevIndex = val.indexs[0]
      this.mainDevOptions[this.mainDevIndex].forcedCodeScan = this.list[this.mainDevIndex].forcedCodeScan //切换主设备的时候也需要扫码，所以切换的时候要把数据给设置回去
      this.mainShow = false
      this.popupShow = false
      this.partEquipment = this.list[this.mainDevIndex].secondaryDevList[0].secondaryDevName
      this.partpmentAllNum = this.list[this.mainDevIndex].secondaryDevList.length

      // 循环当前主设备，获取全部观测点数量
      let array = []
      let arrList = this.list[this.mainDevIndex].secondaryDevList
      for (let n in arrList) {
        let check = arrList[n].checkPointList
        for (let t in check) {
          array.push(check[t])
        }

        // this.list[this.mainDevIndex].secondaryDevOptions = this.list[this.mainDevIndex].secondaryDevList.map(item=>{
        // 	return{
        // 		label:item.secondaryDevName,
        // 		mainDevId:item.mainDevId,
        // 		total: check.length
        // 	}
        // })
      }
      this.allcheckPointNum = array.length
      // 如果当前设备下的测点没有全部完成，把当前设备下已经完成的测点设置为修改状态
      this.list[this.mainDevIndex].isEdit = this.getCompleteNum == 0
      this.list[this.mainDevIndex].isEditList = this.getDevCompleteNum(this.mainDevIndex).ids //把没有数据的记录起来
      // 强制扫码优先级低于防作弊
      if (this.mainDevOptions[this.mainDevIndex].forcedCodeScan === '1'&&this.list[this.mainDevIndex].isEdit && this.mainDevOptions[this.mainDevIndex].forcedNfcCheck === '0') {
        this.showScanTip = true
      }
      // 防作弊
      if (this.mainDevOptions[this.mainDevIndex].isNfcView && this.mainDevOptions[this.mainDevIndex].mainDevCode !== this.options.originDevCode && this.showSubmit && this.list[this.mainDevIndex].status && this.mainDevOptions[this.mainDevIndex].uploadNum == 0) {
        this.isNfcView = true
        this.dialogNfcTip()
      }

      this.equipmentNowState = this.equipmentState.find(item => item.dicValue === this.list[this.mainDevIndex].exemptionStatus).state
      this.equipmentNowCode = this.equipmentState.find(item => item.dicValue === this.list[this.mainDevIndex].exemptionStatus).dicValue
      this.scrollTop = 0 //切换主设备，要把scroll-view划回顶部
      this.partDevIndex = 0
    },
    dialogNfcTip() {
      // 过期或不可编辑
      if(!this.list[this.mainDevIndex].isEdit  || !this.showSubmit) return;
      this.showNFCTip = true
      this.NFCTipText =  this.$$t('该设备需要nfc感应进行点检，点击继续进行nfc感应操作，或者点击拍照进行点检（需上传现场照片）!');
      
      if(this.saveType == 'batchUpload'){//批量编辑跳转需要nfc的第一个设备NFCConfirm
        if(!this.mainDevOptions[this.mainDevIndex].isNfcView){//当前设备不符合跳转
          let index = this.mainDevOptions.findIndex(item => (item.status && item.uploadNum == 0 && item.isNfcView))
          if(index != -1) this.mainConfirm({ indexs: [index] })
        }
      }
    },
    // 部分设备选择
    partConfirm(val) {
      this.partDevIndex = val.indexs[0]
      this.toView = null //先置为空，等dom加载完之后再给toView赋值。避免用户手动滑动之后，此函数无法切换到第一个分部设备
      this.$nextTick(() => {
        // console.log(val, '分布设备名称')
        this.partEquipment = val.value[0].label
        this.partShow = false
        this.popupShow = false
        if (val.indexs[0] == 0) {
          this.toView = 'stateSetDetault'
        } else {
          this.toView = 'details' + val.indexs[0]
        }
        // console.log(this.toView)
      })
      // debugger
    },
    partCancel() {
      this.partShow = false
      this.popupShow = false
    },
    setId(val) {
      // TODO：附件显示问题，备注回显问题
      console.log(val)
      this.itemId = val.plandetailId
      this.fileGroupId = val.fileGroupId
      this.planDetail = val
      this.$refs.enclosureAction.open()
    },
    selectClick(val) {
      switch (val) {
        case 4:
          uni.$off('checkpointSendRemark')
          uni.navigateTo({
            url: '/pages/inspect/remarks?id=' + this.itemId + '&createMonth=' + this.planDetail.createMonth,
            success: () => {
              uni.$once('checkpointSendRemark', info => {
                this.planDetail.remark = info
              })
            }
          })
          break
        case 2:
          uni.navigateTo({
            url: `/pages/inspect/soundRecording?id=${this.itemId}&businessType=${this.businessType}&fileGroupId=${this.fileGroupId}`
          })
          break
        case 3:
          this.video(this.itemId)
          break
        case 1:
          this.photograph(this.itemId)
          break
      }
      this.$refs.enclosureAction.close()
    },
    ulShow(dom) {
      if (this.isView) {
        return false
      }
      const html = dom.target
      if (html.y > 400) {
        this.alarmTop = html.y - 225
        this.alarmLeft = html.x - 90
      } else {
        this.alarmTop = html.y + 25
        this.alarmLeft = html.x - 90
      }
      if (this.alarmLeft < 10) {
        this.alarmLeft = 10
      }
      this.popupShow = true
      this.equipmentStateShow = !this.equipmentStateShow
    },
    stateChange(val) {
      if (this.equipmentNowCode != val.dicValue) {
        this.$set(this.mainDevOptions[this.mainDevIndex], 'isChange', true) //修改了免检状态，变为未上传数据
        this.initMainDevInfo(this.mainDevIndex) //切换免检状态的时候，把当前主设备的数据都去掉
      }
      this.equipmentNowState = val.state
      this.equipmentNowCode = val.dicValue
      this.list[this.mainDevIndex].exemptionStatus = val.dicValue

      if (val.dicValue != 0) {
        this.list[this.mainDevIndex].status = false
        this.list[this.mainDevIndex].isNfcView = true//修改了免检状态,无需再nfc
        this.mainDevOptions[this.mainDevIndex].status = false
        this.mainDevOptions[this.mainDevIndex].isNfcView = true
      } else {
        this.list[this.mainDevIndex].status = true
        this.list[this.mainDevIndex].isNfcView = null
        this.mainDevOptions[this.mainDevIndex].status = true
        this.mainDevOptions[this.mainDevIndex].isNfcView = null
      }

      this.equipmentStateShow = false
      this.popupShow = false
      if (this.equipmentNowCode != 0) {
        //如果是免检状态就当作全部完成
        //免检状态下修改提交时间
        this.list[this.mainDevIndex].secondaryDevList = this.list[this.mainDevIndex].secondaryDevList.map(v => {
          const checkPointList = v.checkPointList.map(vv => {
            return {
              ...vv,
              measDate: moment().format('YYYY-MM-DD HH:mm:ss')
            }
          })
          return {
            ...v,
            checkPointList
          }
        })
        console.log(val, this.list[this.mainDevIndex], 'yyyyyyyyyyyyyyyyy')
        //this.list[this.mainDevIndex].measDate = moment().format('YYYY-MM-DD HH:mm:ss');

        this.mainDevOptions.forEach(v => {
          if (v.mainDevId == this.list[this.mainDevIndex].mainDevId) {
            this.$set(v, 'complete', v.total)
          }
        })
      } else {
        this.inputChange() //如果不是免检状态需要重新计算完成数
      }
    },
    initMainDevInfo(mainDevIndex) {
      //重置当前主设备下的数据（用于切换免检状态的时候）
      this.list[mainDevIndex].isEdit = true
      this.list[mainDevIndex].secondaryDevList.forEach(v => {
        v.checkPointList.forEach(checkPoint => {
          this.$set(checkPoint, 'state', undefined)
          this.$set(checkPoint, 'showOther', false)
          this.$set(checkPoint, 'otherInfo', { description: '', alarmLv: 1 })
          this.$set(checkPoint, 'checkedError', [])
          if (checkPoint.pointType !== '0') {
            this.$set(checkPoint.pointObservation[0], 'value', null)
          }
        })
      })
    },
    //点检录入实际值信息状态颜色展示
    val2alarmColor(row) {
      if (row.pointType != 0 && row.pointObservation[0].value != undefined) {
        // row.alarmLimit.alarmType = "2";
        //* 首先判断类型
        //* 再判断输入框的值处于哪个区间
        /**
         * @alarmType 0 水平超限
         * @alarmType 1 水平低限
         * @alarmType 2 窗内
         * @alarmType 3 窗外
         * @alarmType 4 水平超限等于
         * @alarmType 5 水平底限等于
         * @alarmType 6 等于
         * @alarmType 7 不等于
         * @alarmType 8 窗内等于
         * @alarmType 9 窗外等于
         */
        const type = row.alarmLimit.alarmType
        const value = Number(row.pointObservation[0].value)
        const limit = row.alarmLimit
        const first = limit.first
        const second = limit.second
        const third = limit.third
        const fourth = limit.fourth
        const obj = {
          ...first,
          ...second,
          ...third,
          ...fourth
        }
        console.log('obj', type, obj)

        switch (type) {
          // 水平超限 firstValue < secondValue < thirdValue < fourthValue
          case '0': {
            let result
            if (obj.firstLevel != '0' && obj.firstValue && value > obj.firstValue) {
              result = this.alarmLevelList[parseInt(obj.firstLevel) - 1]?.alarmColor
            }
            if (obj.secondLevel != '0' && obj.secondValue && value > obj.secondValue) {
              result = this.alarmLevelList[parseInt(obj.secondLevel) - 1]?.alarmColor
            }
            if (obj.thirdLevel != '0' && obj.thirdValue && value > obj.thirdValue) {
              result = this.alarmLevelList[parseInt(obj.thirdLevel) - 1]?.alarmColor
            }
            if (obj.fourthLevel != '0' && obj.fourthValue && value > obj.fourthValue) {
              result = this.alarmLevelList[parseInt(obj.fourthLevel) - 1]?.alarmColor
            }
            return result || '#303133'
            break
          }
          // 水平低限 firstValue > secondValue > thirdValue > fourthValue
          case '1': {
            let result
            if (obj.firstLevel != '0' && obj.firstValue && value < obj.firstValue) {
              result = this.alarmLevelList[parseInt(obj.firstLevel) - 1]?.alarmColor
            }
            if (obj.secondLevel != '0' && obj.secondValue && value < obj.secondValue) {
              result = this.alarmLevelList[parseInt(obj.secondLevel) - 1]?.alarmColor
            }
            if (obj.thirdLevel != '0' && obj.thirdValue && value < obj.thirdValue) {
              result = this.alarmLevelList[parseInt(obj.thirdLevel) - 1]?.alarmColor
            }
            if (obj.fourthLevel != '0' && obj.fourthValue && value < obj.fourthValue) {
              result = this.alarmLevelList[parseInt(obj.fourthLevel) - 1]?.alarmColor
            }
            return result || '#303133'
            break
          }
          // 窗内 第一个窗范围需要包含第二个窗
          case '2': {
            let result
            if (obj.firstValue && obj.secondValue && value > obj.firstValue && value < obj.secondValue) {
              result = this.alarmLevelList[parseInt(obj.firstLevel) - 1]?.alarmColor
            }
            if (obj.thirdValue && obj.fourthValue && value > obj.thirdValue && value < obj.fourthValue) {
              result = this.alarmLevelList[parseInt(obj.thirdLevel) - 1]?.alarmColor
            }
            return result || '#303133'
            break
          }
          // 窗外 第一个穿包含第二个窗
          case '3': {
            let result
            if (obj.firstValue && value < obj.firstValue) {
              result = this.alarmLevelList[parseInt(obj.firstLevel) - 1]?.alarmColor
            }
            if (obj.secondValue && value > obj.secondValue) {
              result = this.alarmLevelList[parseInt(obj.secondLevel) - 1]?.alarmColor
            }

            if (obj.thirdValue && value < obj.thirdValue) {
              result = this.alarmLevelList[parseInt(obj.thirdLevel) - 1]?.alarmColor
            }
            if (obj.fourthValue && value > obj.fourthValue) {
              result = this.alarmLevelList[parseInt(obj.fourthLevel) - 1]?.alarmColor
            }
            return result || '#303133'
            break
          }
          // 水平超限等于 firstValue < secondValue < thirdValue < fourthValue
          case '4': {
            let result
            if (obj.firstLevel != '0' && obj.firstValue && value >= obj.firstValue) {
              result = this.alarmLevelList[parseInt(obj.firstLevel) - 1]?.alarmColor
            }
            if (obj.secondLevel != '0' && obj.secondValue && value >= obj.secondValue) {
              result = this.alarmLevelList[parseInt(obj.secondLevel) - 1]?.alarmColor
            }
            if (obj.thirdLevel != '0' && obj.thirdValue && value >= obj.thirdValue) {
              result = this.alarmLevelList[parseInt(obj.thirdLevel) - 1]?.alarmColor
            }
            if (obj.fourthLevel != '0' && obj.fourthValue && value >= obj.fourthValue) {
              result = this.alarmLevelList[parseInt(obj.fourthLevel) - 1]?.alarmColor
            }
            return result || '#303133'
            break
          }
          // 水平低限等于 firstValue > secondValue > thirdValue > fourthValue
          case '5': {
            let result
            if (obj.firstLevel != '0' && obj.firstValue && value <= obj.firstValue) {
              result = this.alarmLevelList[parseInt(obj.firstLevel) - 1]?.alarmColor
            }

            if (obj.secondLevel != '0' && obj.secondValue && value <= obj.secondValue) {
              result = this.alarmLevelList[parseInt(obj.secondLevel) - 1]?.alarmColor
            }

            if (obj.thirdLevel != '0' && obj.thirdValue && value <= obj.thirdValue) {
              result = this.alarmLevelList[parseInt(obj.thirdLevel) - 1]?.alarmColor
            }

            if (obj.fourthLevel != '0' && obj.fourthValue && value <= obj.fourthValue) {
              result = this.alarmLevelList[parseInt(obj.fourthLevel) - 1]?.alarmColor
            }
            return result || '#303133'
            break
          }
          // 等于
          case '6': {
            if (obj.firstValue && value == obj.firstValue) {
              return this.alarmLevelList[parseInt(obj.firstLevel) - 1]?.alarmColor
            }
            return '#303133'
            break
          }
          // 不等于
          case '7': {
            if (obj.firstValue && value != obj.firstValue) {
              return this.alarmLevelList[parseInt(obj.firstLevel) - 1]?.alarmColor
            }
            return '#303133'
            break
          }
          // 窗内等于 第一个窗范围需要包含第二个窗
          case '8': {
            let result
            if (obj.firstValue && obj.secondValue && value >= obj.firstValue && value <= obj.secondValue) {
              result = this.alarmLevelList[parseInt(obj.firstLevel) - 1]?.alarmColor
            }

            if (obj.thirdValue && obj.fourthValue && value >= obj.thirdValue && value <= obj.fourthValue) {
              result = this.alarmLevelList[parseInt(obj.thirdLevel) - 1]?.alarmColor
            }

            return result || '#303133'
            break
          }
          // 窗外等于 第一个穿包含第二个窗
          case '9': {
            let result
            if (obj.firstValue && value <= obj.firstValue) {
              result = this.alarmLevelList[parseInt(obj.firstLevel) - 1]?.alarmColor
            }
            if (obj.secondValue && value >= obj.secondValue) {
              result = this.alarmLevelList[parseInt(obj.secondLevel) - 1]?.alarmColor
            }

            if (obj.thirdValue && value <= obj.thirdValue) {
              result = this.alarmLevelList[parseInt(obj.thirdLevel) - 1]?.alarmColor
            }
            if (obj.fourthValue && value >= obj.fourthValue) {
              result = this.alarmLevelList[parseInt(obj.fourthLevel) - 1]?.alarmColor
            }
            return result || '#303133'
            break
          }
        }
      } else {
        return '#303133'
      }
    },
    alarmLevelChange(val) {
      console.log(val)
      if (this.alarmSelectValue != val.id) {
        this.$set(this.mainDevOptions[this.mainDevIndex], 'isChange', true) //修改了其他的报警等级，变为未上传数据
      }
      this.alarmSelectValue = val.id
      this.list[this.index1].secondaryDevList[this.index2].checkPointList[this.index3].otherInfo.alarmLv = val.id
    },
    alarmLevelDic(lv) {
      for (let i = 0; i < this.alarmLevelList.length; i++) {
        const item = this.alarmLevelList[i]
        if (item.id == lv) {
          return item.alarmName
        }
      }
    },
    alarmColor(lv) {
      for (let i = 0; i < this.alarmLevelList.length; i++) {
        const item = this.alarmLevelList[i]
        if (item.id == lv) {
          return item.alarmColor
        }
      }
    },
    // 单选切换
    runStateChange(val) {
      // console.log(val, '11111')
    },
    radioChange(val, checkPoint) {
      this.$set(this.mainDevOptions[this.mainDevIndex], 'isChange', true) //修改了运行状态，变为未上传数据
      if (checkPoint.state != val) {
        this.$set(checkPoint, 'state', val)
        this.notFilledArr = this.notFilledArr.filter(v => v != checkPoint.id) //选择这里把该测点去掉
        if (val == this.$$t('运行正常')) {
          //选择了正常要把异常的数据都取消掉
          this.$set(checkPoint, 'checkedError', [])
          this.$set(checkPoint, 'showOther', false)
          this.$set(checkPoint, 'otherInfo', { description: '', alarmLv: 1 })
        }
      } else {
        this.$set(checkPoint, 'state', undefined) //取消选中
        if (checkPoint.collectStatus == 2 || checkPoint.collectStatus == 3) {
          //已采集数据,默认为原来的值
          setTimeout(() => {
            this.$set(checkPoint, 'state', checkPoint.stateUp)
            if (checkPoint.stateUp == this.$$t('运行正常')) {
              //选择了正常要把异常的数据都取消掉
              this.$set(checkPoint, 'checkedError', [])
              this.$set(checkPoint, 'showOther', false)
              this.$set(checkPoint, 'otherInfo', { description: '', alarmLv: 1 })
            } else {
              this.$set(checkPoint, 'checkedError', checkPoint.checkedErrorUp)
              this.$set(checkPoint, 'showOther', checkPoint.showOtherUp)
              this.$set(checkPoint, 'otherInfo', checkPoint.otherInfoUp)
            }
          }, 800)
        }
        this.notFilledArr.push(checkPoint.id)
      }
      //修改点检任务上传时间
      this.$set(checkPoint, 'measDate', moment().format('YYYY-MM-DD HH:mm:ss'))
      this.inputChange() //更新切换主设备弹出的“完成数”
      console.log(val, '单选数据', checkPoint)
    },
    checkboxChange(val, pointObservation, checkPoint) {
      console.log('change', val, pointObservation)
      //修改点检任务上传时间
      this.$set(checkPoint, 'measDate', moment().format('YYYY-MM-DD HH:mm:ss'))
      checkPoint.showOther = false
      // delete checkPoint.otherInfo
      for (let i = 0; i < val.length; i++) {
        const id = val[i]
        let other = pointObservation[pointObservation.length - 1]
        if (other.id === id) {
          checkPoint.showOther = true
          this.$set(checkPoint, 'otherInfo', checkPoint.otherInfo ? checkPoint.otherInfo : { description: '', alarmLv: 1 })
        }
      }
      this.notFilledArr = this.notFilledArr.filter(v => v != checkPoint.id) //只要有选择就不能取消了，所以在选择这里把该测点去掉
      this.$forceUpdate()
    },
    getChooseId(errorOption, checkPoint) {
      this.$set(this.mainDevOptions[this.mainDevIndex], 'isChange', true) //修改了观察量，变为未上传数据
      const i = checkPoint.checkedError.findIndex(v => v == errorOption.id)
      if (i != -1) {
        checkPoint.checkedError.splice(i, 1)
      } else {
        this.$set(checkPoint, 'state', this.$$t('运行异常')) //选择了观察量要把状态设置为异常
        checkPoint.checkedError.push(errorOption.id)
        this.notFilledArr = this.notFilledArr.filter(v => v != checkPoint.id) //选择这里把该测点去掉
      }
      console.log(checkPoint.checkedError, i)
      this.checkboxChange(checkPoint.checkedError, checkPoint.pointObservation, checkPoint)
    },
    descriptionChange(checkPoint) {
      //异常情况修改
      //修改点检任务上传时间
      this.$set(checkPoint, 'measDate', moment().format('YYYY-MM-DD HH:mm:ss'))
      if (hasEmoji(checkPoint.otherInfo.description)) {
        //过滤表情
        this.$nextTick(() => {
          this.$set(checkPoint.otherInfo, 'description', filterEmoji(checkPoint.otherInfo.description))
        })
      }
      this.$set(this.mainDevOptions[this.mainDevIndex], 'isChange', true) //修改了异常情况，变为未上传数据
      if (checkPoint.otherInfo.description) {
        this.notFilledArr = this.notFilledArr.filter(v => v != checkPoint.id)
      }
    },
    onBlur(checkPoint) {
      // input失焦判空回填
      if (checkPoint.pointType != '0' && !checkPoint.pointObservation[0].value) {
        setTimeout(() => {
          checkPoint.pointObservation[0].value = checkPoint.pointObservation[0].valueUp
        }, 200)
      }
    },
    inputChange(checkPoint) {
      //输入框修改
      if (checkPoint && checkPoint.pointObservation[0].value) {
        //别的地方也会调用这个函数，所以不一定会有checkPoint
        this.$set(this.mainDevOptions[this.mainDevIndex], 'isChange', true) //修改了输入框的数据，变为未上传数据
        this.notFilledArr = this.notFilledArr.filter(v => v != checkPoint.id)
      }
      // console.log(data.inputValues)
      // console.log(this.mainDevOptions)
      let list = this.mainDevOptions[this.mainDevIndex]
      let arr = []
      let arrList = this.list[this.mainDevIndex].secondaryDevList
      // debugger
      for (let n in arrList) {
        let check = arrList[n].checkPointList
        for (let t in check) {
          //修改数据提交时间
          if (checkPoint && check[t].id == checkPoint.id && !this.list[this.mainDevIndex].secondaryDevList[n].checkPointList[t].measDate) {
            this.list[this.mainDevIndex].secondaryDevList[n].checkPointList[t].measDate = moment().format('YYYY-MM-DD HH:mm:ss')
          }
          if (check[t].pointObservation) {
            if (check[t].pointType === '0') {
              //选择运行正常和运行异常
              if (check[t].state === undefined) {
                arr.push(check[t].pointObservation[0])
              }
            } else {
              //填写观察量
              if (check[t].pointObservation[0].value === null || check[t].pointObservation[0].value === '') {
                arr.push(check[t].pointObservation[0])
              }
            }
          }
        }
      }

      list.complete = list.total - arr.length
      // debugger
      // if (this.inputId === data.id && this.inputVal === '') {
      // 	return
      // }
      // this.inputId = data.id
      // // debugger
      // if (data.inputValues !== '') {
      // 	arr.complete ++
      // } else {
      // 	arr.complete --
      // }
      // let arr = []
      // let arrList = this.list[this.mainDevIndex].secondaryDevList
      // // debugger
      // for (let n in arrList) {
      // 	let check = arrList[n].checkPointList
      // 	for (let t in check) {
      // 		if (check[t].inputValues !== undefined && check[t].inputValues === '') {
      // 			arr.push(check[t].inputValues)
      // 		}
      // 	}

      // }
      // return this.allcheckPointNum - arr.length
    },
    // 上一个
    theLast() {
      if (this.mainDevIndex <= 0) {
        return
      }
      this.mainDevIndex--
      this.mainDevOptions[this.mainDevIndex].forcedCodeScan = this.list[this.mainDevIndex].forcedCodeScan //切换主设备的时候也需要扫码，所以切换的时候要把数据给设置回去
      this.partEquipment = this.list[this.mainDevIndex].secondaryDevList[0].secondaryDevName
      console.log(this.list[this.mainDevIndex], '上一个主设备数据')

      let array = []
      // 循环当前主设备，获取全部观测点数量
      let arrList = this.list[this.mainDevIndex].secondaryDevList
      for (let n in arrList) {
        let check = arrList[n].checkPointList
        for (let t in check) {
          array.push(check[t])
        }
      }
      this.allcheckPointNum = array.length
      // 如果当前设备下的测点没有全部完成，把当前设备下已经完成的测点设置为修改状态
      this.list[this.mainDevIndex].isEdit = this.getCompleteNum == 0
      this.list[this.mainDevIndex].isEditList = this.getDevCompleteNum(this.mainDevIndex).ids //把没有数据的记录起来
      if (this.mainDevOptions[this.mainDevIndex].forcedCodeScan === '1' && this.list[this.mainDevIndex].isEdit) {
        this.showScanTip = true
      }
      // 防作弊
      if (this.mainDevOptions[this.mainDevIndex].isNfcView && this.mainDevOptions[this.mainDevIndex].mainDevId !== this.option.devId && this.showSubmit && this.list[this.mainDevIndex].status) {
        this.isNfcView = true
        this.dialogNfcTip()
      }
      this.equipmentNowState = this.equipmentState.find(item => item.dicValue === this.list[this.mainDevIndex].exemptionStatus).state
      this.equipmentNowCode = this.equipmentState.find(item => item.dicValue === this.list[this.mainDevIndex].exemptionStatus).dicValue
      this.scrollTop = 0 //切换主设备，要把scroll-view划回顶部
      setTimeout(() => {
        //避免此时正在滑动，所以要再一次复位
        this.scrollTop = 0 //切换主设备，要把scroll-view划回顶部
      }, 300)
    },
    //下一个
    theNext() {
      if (this.mainDevIndex >= this.list.length - 1) {
        return
      }
      this.mainDevIndex++
      this.mainDevOptions[this.mainDevIndex].forcedCodeScan = this.list[this.mainDevIndex].forcedCodeScan //切换主设备的时候也需要扫码，所以切换的时候要把数据给设置回去
      this.partEquipment = this.list[this.mainDevIndex].secondaryDevList[0].secondaryDevName
      let array = []
      // 循环当前主设备，获取全部观测点数量
      let arrList = this.list[this.mainDevIndex].secondaryDevList
      for (let n in arrList) {
        let check = arrList[n].checkPointList
        for (let t in check) {
          array.push(check[t])
        }
      }
      this.allcheckPointNum = array.length
      // 如果当前设备下的测点没有全部完成，把当前设备下已经完成的测点设置为修改状态
      this.list[this.mainDevIndex].isEdit = this.getCompleteNum == 0
      this.list[this.mainDevIndex].isEditList = this.getDevCompleteNum(this.mainDevIndex).ids //把没有数据的记录起来
      if (this.mainDevOptions[this.mainDevIndex].forcedCodeScan === '1' && this.list[this.mainDevIndex].isEdit) {
        this.showScanTip = true
      }
      // 防作弊
      if (this.mainDevOptions[this.mainDevIndex].isNfcView && this.mainDevOptions[this.mainDevIndex].mainDevId !== this.option.devId && this.showSubmit && this.list[this.mainDevIndex].status) {
        this.isNfcView = true
        this.dialogNfcTip()
      }
      this.equipmentNowState = this.equipmentState.find(item => item.dicValue === this.list[this.mainDevIndex].exemptionStatus).state
      this.equipmentNowCode = this.equipmentState.find(item => item.dicValue === this.list[this.mainDevIndex].exemptionStatus).dicValue
      this.scrollTop = 0 //切换主设备，要把scroll-view划回顶部
      setTimeout(() => {
        //避免此时正在滑动，所以要再一次复位
        this.scrollTop = 0 //切换主设备，要把scroll-view划回顶部
      }, 300)
    },
    // 上传
    save(type, mainDevIndex) {
      // 当前主设备需要扫码
      console.log(this.$store.state.list, this.values, this.ids, '当前设备数据', '当前步骤号->', this.mainDevIndex, 'ckck->', this.list)
      // let obj = {exemptionStatus: ["0"],values: [1], ids: [284901],dataSource:'app',otherList: [{
      //     id:49223122,
      //     value:'1',
      //     exemptionStatus:'0',
      //     alarmLv:"2",
      //     description:""
      // }]}
      //     this.request(obj);
      let isError = false //是否有错误，如果有错误就不给提交
      if (type == 'batchUpload') {
        if (this.mainDevOptions[mainDevIndex].isNfcView && this.list[mainDevIndex].status) {
          this.dialogNfcTip()
          return
        }
        //如果是批量提交，需要循环处理
        if (this.mainDevOptions[mainDevIndex].forcedCodeScan === '1' && this.getDevCompleteNum(mainDevIndex).completeNum > 0 && this.list[this.mainDevIndex].status) {
          this.mainConfirm({ indexs: [mainDevIndex] })
          this.showScanTip = true
          isError = true
          return false
        }
      } else {
        if (this.mainDevOptions[this.mainDevIndex].isNfcView && this.mainDevOptions[this.mainDevIndex].mainDevCode !== this.options.originDevCode && this.showSubmit && this.list[this.mainDevIndex].status && this.mainDevOptions[this.mainDevIndex].uploadNum == 0) {
          this.dialogNfcTip()
          return
        }
        if (this.mainDevOptions[this.mainDevIndex].forcedCodeScan === '1' && this.getDevCompleteNum(this.mainDevIndex).completeNum > 0) {
          this.showScanTip = true
          isError = true
          return false
        }
      }
      // debugger
      let exemptionStatus = []
      let measDateList = []
      let values = []
      let gatherValues = []
      let ids = []
      let alarmLvArr = []
      let otherList = []
      let completeCheckPointIds = [] //记录当前设备下哪些测点已完成
      let remarks = []
      let equipmentNowCode = this.equipmentNowCode

      // this.list.forEach((v) => {
      //   v.secondaryDevList.forEach((vv) => {
      //     vv.checkPointList.forEach((vvv) => {
      //       if (!vvv.attachmentKey || vvv.attachmentKey == "null") {
      //         vvv.attachmentKey = [];
      //       }
      //       vvv.attachmentKey = vvv.attachmentKey.join(",");
      //     });
      //   });
      // });

      // 判断当前主设备的status是什么状态，再push
      let arr = this.list[this.mainDevIndex]
      if (type == 'batchUpload') {
        //如果是批量提交，需要循环处理
        arr = this.list[mainDevIndex]
        equipmentNowCode = this.equipmentState.find(item => item.dicValue === this.list[mainDevIndex].exemptionStatus).dicValue
      }
      console.log(213123, arr)
      // 判断主设备的status是否是运行（true）状态
      if (arr.status === true) {
        let second = arr.secondaryDevList
        for (let k in second) {
          let check = second[k].checkPointList

          console.log(check, 111111)
          for (let j in check) {
            //添加：采样时间
            for (let r in check[j].pointObservation) {
              measDateList.push(check[j].measDate)
            }
            remarks.push(check[j].remark)
            this.notFilledArr.push(check[j].id) //保存的时候把所有的测点id存起来，当作没填写。在后续如果有填写就去掉（尽量不该原代码的方法）
            // 如果选择其他
            if (check[j].checkedError) {
              if (check[j].state == this.$$t('运行异常') && check[j].checkedError.length == 0) {
                this.$refs.layout.showToast({
                  title: this.$$t('请选择至少一个异常描述'),
                  icon: 'error'
                })
                isError = true
                return false
              }
              if (check[j].checkedError.some(item => item === check[j].pointObservation[check[j].pointObservation.length - 1].id) && check[j].state !== this.$$t('运行正常')) {
                //
                if (!check[j].otherInfo.description) {
                  this.$refs.layout.showToast({
                    title: this.$$t('请输入异常情况'),
                    icon: 'error'
                  })
                  isError = true
                  return false
                }
                otherList.push({
                  id: check[j].pointObservation[check[j].pointObservation.length - 1].id,
                  value: '1',
                  exemptionStatus: check[j].exemptionStatus,
                  alarmLv: check[j].otherInfo.alarmLv,
                  description: check[j].otherInfo.description
                })
                //把当前测点的采集状态设置为已采集3，搜索的时候就可以修改显示颜色---start
                check[j].collectStatus = 3
                //把当前测点的采集状态设置为已采集3，搜索的时候就可以修改显示颜色---end
              }
              if (check[j].otherInfo && check[j].state === this.$$t('运行正常')) {
                otherList.push({
                  id: check[j].pointObservation[check[j].pointObservation.length - 1].id,
                  value: '0',
                  exemptionStatus: check[j].exemptionStatus,
                  alarmLv: check[j].otherInfo.alarmLv,
                  description: check[j].otherInfo.description
                })
              }
              // debugger
              if (check[j].state == this.$$t('运行正常')) {
                //同时选择运行正常和观察量，就传运行正常
                this.notFilledArr = this.notFilledArr.filter(v => v != check[j].id)
                completeCheckPointIds.push(check[j].id)
                exemptionStatus.push(parseInt(equipmentNowCode))
                ids.push(check[j].pointObservation[0].id)
                alarmLvArr.push(check[j].pointObservation[0].alarmLv)
                values.push(1)
                gatherValues.push(undefined) //代表当前测点没有数采标签
                //把当前测点的采集状态设置为已采集3，搜索的时候就可以修改显示颜色---start
                check[j].collectStatus = 3
                //把当前测点的采集状态设置为已采集3，搜索的时候就可以修改显示颜色---end
                for (let r in check[j].pointObservation) {
                  //所有测点都需要传
                  if (check[j].pointObservation[r].observationCode !== '0') {
                    exemptionStatus.push(parseInt(equipmentNowCode))
                    ids.push(check[j].pointObservation[r].id)
                    alarmLvArr.push(check[j].pointObservation[r].alarmLv)
                    values.push(0)
                    gatherValues.push(undefined) //代表当前测点没有数采标签
                  }
                }
              }
              if (check[j].checkedError && check[j].checkedError.length > 0 && check[j].state == this.$$t('运行异常')) {
                this.notFilledArr = this.notFilledArr.filter(v => v != check[j].id)
                completeCheckPointIds.push(check[j].id)
                let checkList = check[j].checkedError
                // for (let r in checkList) {
                // 	exemptionStatus.push(parseInt(equipmentNowCode))
                // 	ids.push(checkList[r])
                // 	values.push(1)
                // }
                for (let r in check[j].pointObservation) {
                  //所有测点都需要传
                  exemptionStatus.push(parseInt(equipmentNowCode))
                  ids.push(check[j].pointObservation[r].id) //所有测点都需要传
                  alarmLvArr.push(check[j].pointObservation[r].alarmLv)
                  if (check[j].pointObservation[r].observationCode === '0') {
                    //如果该测点是“运行正常”
                    if (check[j].state === this.$$t('运行正常')) {
                      //如果选择了正常，就要把测点设为1
                      values.push(1)
                      gatherValues.push(undefined) //代表当前测点没有数采标签
                      //把当前测点的采集状态设置为已采集3，搜索的时候就可以修改显示颜色---start
                      check[j].collectStatus = 3
                      //把当前测点的采集状态设置为已采集3，搜索的时候就可以修改显示颜色---end
                    } else {
                      values.push(0)
                      gatherValues.push(undefined) //代表当前测点没有数采标签
                    }
                  } else {
                    if (checkList.includes(check[j].pointObservation[r].id)) {
                      //如果有被勾选
                      if (check[j].state === this.$$t('运行正常')) {
                        //如果选择了正常，就要把
                        values.push(0)
                        gatherValues.push(undefined) //代表当前测点没有数采标签
                      } else {
                        values.push(1)
                        gatherValues.push(undefined) //代表当前测点没有数采标签
                        //把当前测点的采集状态设置为已采集3，搜索的时候就可以修改显示颜色---start
                        check[j].collectStatus = 3
                        //把当前测点的采集状态设置为已采集3，搜索的时候就可以修改显示颜色---end
                      }
                    } else {
                      //如果没被勾选，就赋值为0
                      values.push(0)
                      gatherValues.push(undefined) //代表当前测点没有数采标签
                    }
                  }
                }
              }
            }
            if (check[j].inputValues !== undefined && check[j].pointObservation[0].value !== '') {
              if (check[j].pointObservation[0].value != null && check[j].pointObservation[0].value != undefined) {
                this.notFilledArr = this.notFilledArr.filter(v => v != check[j].id)
                completeCheckPointIds.push(check[j].id)
                //把当前测点的采集状态设置为已采集3，搜索的时候就可以修改显示颜色---start
                check[j].collectStatus = 3
                //把当前测点的采集状态设置为已采集3，搜索的时候就可以修改显示颜色---end
              }
              exemptionStatus.push(parseInt(equipmentNowCode))
              ids.push(check[j].pointObservation[0].id)
              alarmLvArr.push(check[j].pointObservation[0].alarmLv)
              values.push(check[j].pointObservation[0].value)
              gatherValues.push(check[j].pointObservation[0].gatherValue)
            }
            if (check[j].inputValues !== undefined && check[j].pointObservation[0].value === '') {
              exemptionStatus.push(parseInt(equipmentNowCode))
              ids.push(check[j].pointObservation[0].id)
              alarmLvArr.push(check[j].pointObservation[0].alarmLv)
              values.push(null)
              gatherValues.push(check[j].pointObservation[0].gatherValue)
            }
          }
        }
        this.notFilledArr = [...new Set(this.notFilledArr)] //去重一下
        console.log({
          exemptionStatus: exemptionStatus,
          values: values,
          ids: ids,
          dataSource: 'app',
          otherList
        })

        // const params = {exemptionStatus: exemptionStatus,values: values, ids: ids,dataSource:'app',otherList};
        let dataList = []
        for (let i = 0; i < ids.length; i++) {
          let dataItem = {
            id: ids[i],
            value: values[i],
            alarmLv: alarmLvArr[i],
            exemptionStatus: exemptionStatus[i],
            remark: remarks[i],
            measDate: measDateList[i]
          }
          if (gatherValues[i] !== undefined) {
            dataItem.gatherValue = gatherValues[i] //数采值
            dataItem.dataSource = values[i] == gatherValues[i] ? 'dataAcquisition' : 'app' //数据来源
          }
          dataList.push(dataItem)
        }
        const params = {
          dataSource: 'app',
          planExeId: this.businessId,
          checkType: this.checkType,
          list: dataList,
          completeCheckPointIds: {
            [type == 'batchUpload' ? mainDevIndex : this.mainDevIndex]: completeCheckPointIds
          },
          otherList: otherList
        }
        console.log(isError, 'params-----------------------------------------')
        if (isError) {
          return false
        } //有错误信息，不给提交
        if (type == 'batchUpload') {
          //如果是批量提交就return出去统一处理
          return params
        }
        return params
      }
      if (arr.status !== true) {
        let second = arr.secondaryDevList
        for (let k in second) {
          let check = second[k].checkPointList
          for (let j in check) {
            //添加：采样时间
            for (let r in check[j].pointObservation) {
              measDateList.push(check[j].measDate)
              ids.push(check[j].pointObservation[r].id)
            }
            //把当前测点的采集状态设置为已采集3，搜索的时候就可以修改显示颜色---start
            check[j].collectStatus = 3
            //把当前测点的采集状态设置为已采集3，搜索的时候就可以修改显示颜色---end
            this.notFilledArr = this.notFilledArr.filter(v => v != check.id) //设置为免检则都不需要填写
            completeCheckPointIds.push(check[j].id)
            if (check[j].checkedError && check[j].checkedError.length === 0) {
              // ids.push(...check[j].pointObservation.map(v => v.id))
              alarmLvArr.push(...check[j].pointObservation.map(v => v.alarmLv))
              exemptionStatus.push(...check[j].pointObservation.map(v => parseInt(equipmentNowCode)))
            }
            if (check[j].checkedError && check[j].checkedError.length > 0) {
              // ids.push(check[j].checkedError[0])
              alarmLvArr.push(check[j].pointObservation[0].alarmLv)
              exemptionStatus.push(parseInt(equipmentNowCode))
            }
            if (check[j].pointObservation[0].value !== undefined) {
              // ids.push(...check[j].pointObservation.map(v => v.id))
              alarmLvArr.push(...check[j].pointObservation.map(v => v.alarmLv))
              exemptionStatus.push(...check[j].pointObservation.map(v => parseInt(equipmentNowCode)))
            }
          }
        }
        // const params = {exemptionStatus: exemptionStatus, ids: ids,dataSource:'app'}
        let dataList = []
        for (let i = 0; i < ids.length; i++) {
          dataList.push({
            id: ids[i],
            alarmLv: alarmLvArr[i],
            value: values[i],
            exemptionStatus: exemptionStatus[i],
            remark: remarks[i],
            measDate: measDateList[i]
          })
        }
        const params = {
          dataSource: 'app',
          planExeId: this.businessId,
          checkType: this.checkType,
          list: dataList,
          completeCheckPointIds: {
            [type == 'batchUpload' ? mainDevIndex : this.mainDevIndex]: completeCheckPointIds
          },
          otherList: otherList
        }
        if (type == 'batchUpload') {
          //如果是批量提交就return出去统一处理
          return params
        }
        return params
      }
      console.log(ids, measDateList, values, '0000000000000000000000000000')
    },
    request(params, type) {
      if (type == 'batchUpload') {
        this.mainDevOptions.forEach(v => {
          this.$set(v, 'isChange', false) //提交后，变为已上传数据
        })
      }
      console.info(params.list, 'ttttttttttttttttttttttttttttttttt')
      // params.list = params.list.map(v => {
      //   if (v.value == 1) {
      //     v.measDate = moment().format('YYYY-MM-DD HH:mm:ss')
      //   }
      //   if (v.alarmLv === null && (v.value !== '') && (v.value !== null) && (v.value !== undefined)) {
      //     //观察量
      //     v.measDate = moment().format('YYYY-MM-DD HH:mm:ss')
      //   }
      //   // v.measDate = moment().format('YYYY-MM-DD HH:mm:ss')
      //   return v
      // })
      if (this.equipmentNowCode == 0 && (params.list.length == 0 || params.list.filter(v => v.value).length == 0)) {
        //当前至少要填一个数据才能上传或者批量上传
        return this.$refs.layout.showToast({ title: this.$$t('请录入点检结果'), icon: 'error' })
      }
      let devCompleteNum = this.getDevCompleteNum(this.mainDevIndex)
      console.log(devCompleteNum, this.isNfcView, 'devCompleteNum')
      let needUpload = false
      
      if (!devCompleteNum.isFile) {
        this.$refs.layout.showToast({ title: this.$$t('请上传附件'), icon: 'error', position: 'top' })
        // return
        // 若填写一个测点且附件必填需
        if (devCompleteNum.completeNum == devCompleteNum.list.length || (devCompleteNum.completeNum == 1 && !devCompleteNum.isFile)) {
          this.notFilledArr = devCompleteNum.list.map(item => item.id)
        } else {
          this.notFilledArr = this.notFilledArr.concat(devCompleteNum.list.map(item => item.id))
        }
        return
      }
      

      // if(this.getDevCompleteNum(this.mainDevIndex).isComplete!=true){ //提示当前设备是否有未填写的测点
      //   uni.showModal({
      //     title: this.$$t("提示"),
      //     content: this.$$t("还存在未填写测点,确认提交吗?"),
      //     success: (res)=> {
      //       if (res.confirm) {
      //         if(type=='batchUpload'){
      //           this.mainDevOptions.forEach(v=>{
      //             this.$set(v,'isChange',false) //提交后，变为已上传数据
      //           })
      //         }else{
      //           this.$set(this.mainDevOptions[this.mainDevIndex],'isChange',false) //提交后，变为已上传数据
      //         }
      //         //* 检查网络状态
      //         this.checkNet()
      //           .then(res => {
      //             //* 在线时调用
      //             this.inspect(res, params)
      //             // this.inspect(false, params, Number(this.mainDevIndex + 1));
      //           })
      //           .catch(err => {
      //             //* 离线时调用
      //             this.inspect(err, params, Number(this.mainDevIndex + 1))
      //           })
      //       } else if (res.cancel) {
      //         console.log('用户点击取消');
      //       }
      //     }
      //   });
      // }else{
      //   this.$set(this.mainDevOptions[this.mainDevIndex],'isChange',false) //提交后，变为已上传数据
      // }
      //* 检查网络状态
      this.checkNet()
        .then(res => {
          //* 在线时调用
          this.inspect(res, params, 1, needUpload)
          // this.inspect(false, params, Number(this.mainDevIndex + 1));
        })
        .catch(err => {
          //* 离线时调用
          this.inspect(err, params, Number(this.mainDevIndex + 1))
        })
    },
    checkNet() {
      uni.showLoading({
        title: this.$$t('加载中')
      })
      return new Promise((resolve, reject) => {
        uni.getNetworkType({
          success: res => {
            if (res.networkType === 'none') {
              reject(false)
            } else {
              resolve(true)
            }
            uni.hideLoading()
          }
        })
      })
    },
    inspect(boo = true, params, StepId = 1, needUpload = false) {
      this.$refs.confirmPopup.close()
      if (!params) return
      // 清除当前任务的失败队列
      if (this.$route.query.reUpload) {
        callApi('JimDBhelp', 'deleteDebugForTaskid', [{ taskid: Number(this.$route.query.id) }]).then(res => {
          console.log('失败任务', res)
        })
      }
      if (boo) {
        inspectSendResult(params).then(res => {
          // this.startPullDownRefresh(); //提交之后请求最新数据(不能在此请求最新数据，会导致设备切换错乱：mainDevIndex)
          this.mainDevOptions = this.mainDevOptions.map((v, i) => {
            const completeCheckPointIds = params.completeCheckPointIds[i]
            return {
              ...v,
              uploadNum: completeCheckPointIds ? completeCheckPointIds.length : v.uploadNum //如果只传一个设备的话，就只会传当前设备的数据
            }
          })
          if (this.devAllCompleteNum === this.list.length) {
            this.$refs.successPopup.open('center')
            return
          }
          // 提交更新已采集数据
          let ids = []
          params.list.forEach(item => {
            if (item.measDate) {
              ids.push(item.id)
            }
          })
          this.list = this.list.map(item => {
            item.secondaryDevList = item.secondaryDevList.map(sec => {
              sec.checkPointList = sec.checkPointList.map(che => {
                if (ids.includes(che.id)) {
                  //已采集点状态变更,数据存储
                  che.collectStatusUp = 2
                  if (che.pointType == '0') {
                    che.stateUp = che.state
                    if (che.state != this.$$t('运行正常')) {
                      che.checkedErrorUp = che.checkedError
                      che.showOtherUp = che.showOther
                      che.otherInfoUp = che.otherInfo
                    }
                  } else {
                    che.pointObservation[0].valueUp = che.pointObservation[0].value
                  }
                }
                return che
              })
              return sec
            })
            return item
          })
          console.log(this.list, '提交成功！')
          uni.showToast({ title: this.$$t(needUpload ? '部分提交成功' : '提交成功！'), icon: 'none' })
          if (this.getDevCompleteNum(this.mainDevIndex).isComplete) {
            //当前设备全部填写完成才自动切换
            this.theNext()
          }
          // this.getData(true, this.option)
          console.log(res.result, '发送数据回调')
        })
      } else {
        const requestData = {
          blocks: {
            paramBlock: {
              data: params,
              limit: params.pageSize ?? 20,
              offset: params.pageNum ?? 1,
              blockId: 'paramBlock',
              orderBy: params.orderBy ?? undefined
            }
          },
          fileMsg: this.fileMsg.map(item => {
            return {
              businessId: item.itemId,
              businessType: item.businessType,
              fileUrl: item.fileUrl,
              fileGroupId: item.fileGroupId
            }
          })
        }
        // let jsonData = {"blocks":{"paramBlock":{"data":{"exemptionStatus":[0],"values":[1],"ids":[28054258],"dataSource":"app","otherList":[]},"limit":20,"offset":1,"blockId":"paramBlock"}}}
        let data = {
          methods: 'post',
          // url: 'http://10.101.16.41:7777/prod-api/check/planexe/execute',
          url: window.globalConfig.VUE_APP_BASE_API + '/check/planexe/offline/execute',
          taskid: Number(this.$route.query.id),
          stpeid: StepId,
          status: 0,
          isFile: this.fileMsg.length ? 1 : 0,
          uploadFileUrl: window.globalConfig.VUE_APP_BASE_API,
          uploadTokenUrl: window.globalConfig.VUE_WORKFLOW_SRC,
          data: JSON.stringify(requestData),
          token: this.token,
          DBname: 'eamMobile1'
        }
        console.log(this.token, '==============', data)
        //* 离线存储
        callApi('JimDBhelp', 'addDownloadQueue', [data])
          .then(res => {
            console.log(res, 'addDownloadQueue---,存储成功(离线)')
            this.fileMsg = []
            // this.getData(false, this.option)
          })
          .catch(err => {
            console.log(err)
          })
      }
    },
    // 确认上传数据弹窗
    handleDialog(type, alwaysSubmit, dialogSubmit) {
      //alwaysSubmit是否直接提交不管错误
      console.info(type, '555555555555555555555555555')
      this.$forceUpdate() //强行刷新数据，避免弹窗的textarea的数据不显示
      this.saveType = type //保存提交类型，给弹窗用
      let showPop = false
      if (type == 'batchUpload') {
        this.list.forEach(mainDev => {
          mainDev.secondaryDevList.forEach(secondaryDev => {
            if (this.secondaryDev2abnormalPoint(secondaryDev).length > 0) {
              showPop = true
            }
          })
        })
      } else {
        this.list[this.mainDevIndex].secondaryDevList.forEach(secondaryDev => {
          if (this.secondaryDev2abnormalPoint(secondaryDev).length > 0) {
            showPop = true
          }
        })
      }
      if (alwaysSubmit && !dialogSubmit) {
        console.log('showPop', showPop)
        if (showPop) {
          this.$refs.confirmPopup.open()
        } else {
          this.handleDialog(type, true, true)
        }
        return false
      }

      if (type == 'batchUpload') {
        //如果是批量提交就会把数据返回来一起处理
        let isError = false //是否有错误，如果有错误就不给提交
        let params = {
          dataSource: 'app',
          planExeId: this.businessId,
          checkType: this.checkType,
          list: [],
          completeCheckPointIds: {},
          otherList: []
        }
        for (let i = 0; i < this.list.length; i++) {
          const val = this.save(type, i)
          if (val === false) {
            isError = true
            return false
          } //有错误提示不给提交
          params.list = params.list.concat(val.list)
          params.completeCheckPointIds[i] = val.completeCheckPointIds[i]
          params.otherList = params.otherList.concat(val.otherList)
        }
        if (!dialogSubmit && (isError === true || this.checkParams(params || {}, type, alwaysSubmit) === false)) {
          return false
        } //有错误提示不给提交
        if (showPop && !alwaysSubmit && !dialogSubmit) {
          this.$refs.confirmPopup.open()
        } else {
          this.request(params, type)
        }
        return false
      }
      const params = this.save(type) //不需要确认弹窗，直接上传前判断是否有测点未填写：2023/07/20
      console.log(params, showPop, dialogSubmit, this.checkParams(params, type, alwaysSubmit), 'params5555555555555555555555555555555')
      if (!dialogSubmit && (params === false || this.checkParams(params, type, alwaysSubmit) === false)) {
        return false
      } //有错误提示不给提交
      if (showPop && !alwaysSubmit && !dialogSubmit) {
        this.$refs.confirmPopup.open()
      } else {
        this.request(params, type)
      }
    },
    checkParams(params, type, alwaysSubmit) {
      //alwaysSubmit是否直接提交不管错误
      params.list = (params.list || []).map(v => {
        if (v.value == 1) {
          v.measDate = moment().format('YYYY-MM-DD HH:mm:ss')
        }
        if (v.alarmLv === null && v.value !== '' && v.value !== null && v.value !== undefined) {
          //观察量
          v.measDate = moment().format('YYYY-MM-DD HH:mm:ss')
        }
        // v.measDate = moment().format('YYYY-MM-DD HH:mm:ss')
        return v
      })
      if (this.equipmentNowCode == 0 && (params?.list.length == 0 || params?.list.filter(v => v.value).length == 0)) {
        //当前至少要填一个数据才能上传或者批量上传
        this.$refs.layout.showToast({ title: this.$t('请录入点检结果'), icon: 'error' })
        return false //返回false给其他函数做判断
      }
      let devCompleteNum = this.getDevCompleteNum(this.mainDevIndex)
      console.log(devCompleteNum, this.isNfcView, 'devCompleteNum')
      let needUpload = false
      if (!alwaysSubmit && devCompleteNum.isComplete != true) {
        //提示当前设备是否有未填写的测点
        uni.showModal({
          title: this.$t('提示'),
          content: this.$t('还存在未填写测点,确认提交吗?'),
          success: res => {
            if (res.confirm) {
              if (!devCompleteNum.isFile) {
                this.$refs.layout.showToast({ title: this.$$t('请上传附件'), icon: 'error', position: 'top' })
                // return
                // 若填写一个测点且附件必填需
                if (devCompleteNum.completeNum == devCompleteNum.list.length || (devCompleteNum.completeNum == 1 && !devCompleteNum.isFile)) {
                  this.notFilledArr = devCompleteNum.list.map(item => item.id)
                } else {
                  this.notFilledArr = this.notFilledArr.concat(devCompleteNum.list.map(item => item.id))
                }
                return
              }
              this.handleDialog(this.saveType, true)
              return false
              if (type == 'batchUpload') {
                this.mainDevOptions.forEach(v => {
                  this.$set(v, 'isChange', false) //提交后，变为已上传数据
                })
              } else {
                this.$set(this.mainDevOptions[this.mainDevIndex], 'isChange', false) //提交后，变为已上传数据
              }
              //* 检查网络状态
              this.checkNet()
                .then(res => {
                  //* 在线时调用
                  this.inspect(res, params)
                  // this.inspect(false, params, Number(this.mainDevIndex + 1));
                })
                .catch(err => {
                  //* 离线时调用
                  this.inspect(err, params, Number(this.mainDevIndex + 1))
                })
            } else if (res.cancel) {
              console.log('用户点击取消')
            }
          }
        })
        return false //返回false给其他函数做判断
      }
    },
    handleBack() {
      //返回上一页
      const completeNum = this.mainDevOptions.reduce((a, b) => {
        return a + b.complete
      }, 0) //已完成的数量
      let obj = {}
      obj.id = this.businessId
      obj.list = this.list
      obj.completeNum = completeNum
      this.$store.commit('changeList', obj)
      this.$refs.successPopup.close() //不关闭在h5页面无法滑动
      setTimeout(() => {
        //延迟返回，避免弹窗未关闭导致在h5无法滑动
        // 返回上一页
        uni.navigateBack({
          success: () => {
            // setTimeout(() => {
            //   uni.$emit('inspectComplete', obj)
            // }, 100)
          }
        })
      }, 300) //延迟300毫秒，因为uni-popup组件内部也是延迟300毫秒处理关闭的
      // uni.navigateTo({
      // 	url: '/pages/inspect/index'
      // })
    },
    // 修改
    modify() {
      if (this.mainDevOptions[this.mainDevIndex].forcedCodeScan === '1') {
        this.showScanTip = true
        return
      }
      this.list[this.mainDevIndex].isEdit = true
    },
    popShow(val) {
      // debugger
      console.log(val, '观察点标准内容')
      this.title = val.pointName
      this.content = val.description
      let arr = val.description.split(';')
      if ([6, 7, '6', '7'].includes(val.alarmLimit.alarmType)) {
        //如果报警类型为6,7(等于，不等于)，就只需要取前两条，第一条是类型。第二条才是数据
        arr.splice(2)
      }
      let res = `<div style="font-size:15px;color:#606266;text-align:center;">${arr[0]}</div>`
      const alarmColorObj = {
        1: val.alarmLimit['first'].firstLevel,
        2: val.alarmLimit['second'].secondLevel,
        3: val.alarmLimit['third'].thirdLevel,
        4: val.alarmLimit['fourth'].fourthLevel
      }
      for (let i = 1; i < arr.length; i++) {
        if (alarmColorObj[i] == 0) {
          //如果是0,代表是正常
          res += `<div style="font-size:15px;color:#606266">${arr[i]}</div>`
        } else {
          const alarmColor = this.alarmLevelList[alarmColorObj[i] - 1]?.alarmColor
          res += `<div style="font-size:15px;color:${alarmColor}">${arr[i]}</div>`
        }
      }
      this.content = res
      // this.standardShow = true
    },
    confirm() {
      this.standardShow = false
    },
    // 搜索
    handleSearch() {
      uni.$off('inspectSearchDev')
      uni.setStorageSync('inspectSearchDevList', encodeURIComponent(JSON.stringify(this.list).replace(/%/g, '%25')))
      uni.navigateTo({
        url: `/pages/search/index?type=equipment&page=inspect`,
        success: () => {
          uni.$once('inspectSearchDev', data => {
            let arr = data[0]
            let obj = data[1]
            this.toView = null //先置为空，等dom加载完之后再给toView赋值。避免用户手动滑动之后，此函数无法切换到第一个分部设备
            setTimeout(() => {
              for (let i in arr) {
                // 判断是主设备还是部分设备
                if (obj.rank === 'main' && obj.devName === arr[i].mainDevName) {
                  this.mainDevIndex = parseInt(i)
                  this.partDevIndex = 0
                  this.partEquipment = arr[i].secondaryDevList[0].secondaryDevName
                  this.toView = 'stateSetDetault'
                }
                let second = arr[i].secondaryDevList
                for (let j in second) {
                  if (obj.rank === 'second' && obj.devName === second[j].secondaryDevName) {
                    // debugger
                    this.mainDevIndex = parseInt(i)
                    this.partDevIndex = parseInt(j)
                    this.partEquipment = second[j].secondaryDevName
                    if (parseInt(j) == 0) {
                      this.toView = 'stateSetDetault'
                    } else {
                      this.toView = 'details' + parseInt(j)
                    }
                  }
                }
              }
              let array = []
              // 循环当前主设备，获取全部观测点数量
              let arrList = this.list[this.mainDevIndex].secondaryDevList
              for (let n in arrList) {
                let check = arrList[n].checkPointList
                for (let t in check) {
                  array.push(check[t])
                }
              }
              this.mainDevOptions[this.mainDevIndex].forcedCodeScan = this.list[this.mainDevIndex].forcedCodeScan //切换主设备的时候也需要扫码，所以切换的时候要把数据给设置回去
              this.allcheckPointNum = array.length
              // 如果当前设备下的测点没有全部完成，把当前设备下已经完成的测点设置为修改状态
              this.list[this.mainDevIndex].isEdit = this.getCompleteNum == 0
              this.list[this.mainDevIndex].isEditList = this.getDevCompleteNum(this.mainDevIndex).ids //把没有数据的记录起来
              if (this.mainDevOptions[this.mainDevIndex].forcedCodeScan === '1' && this.list[this.mainDevIndex].isEdit) {
                this.showScanTip = true
              }
            }, 100)
          })
        }
      })
    },
    // 扫码
    scanCode() {
      this.showScanTip = false
      callApi('MideaBarcode', 'scan', [1, 1, 0, 0]).then(res => {
        console.log(res, '扫码回调res')
        const mainDevIndex = this.list.findIndex(item => res.text.includes(item.mainQrCode))
        if (mainDevIndex > -1) {
          this.$refs.uToast.show({
            type: 'success',
            message: this.$$t('扫描成功')
          })
          this.mainDevIndex = mainDevIndex
          this.mainDevOptions[this.mainDevIndex].forcedCodeScan = '0'
          this.list[this.mainDevIndex].isEdit = true
          // 循环当前主设备，获取全部观测点数量
          let array = []
          let arrList = this.list[this.mainDevIndex].secondaryDevList
          for (let n in arrList) {
            let check = arrList[n].checkPointList
            for (let t in check) {
              array.push(check[t])
            }
          }
          this.allcheckPointNum = array.length
          if (this.mainDevIndex != mainDevIndex) {
            this.mainConfirm({ indexs: [mainDevIndex] })
          }
        } else {
          this.showScanTip = true
          this.scanTipText = this.$$t('请扫描正确设备')
        }
      })
    },
    // 扫码弹窗确认
    scanConfirm() {
      // this.scanCancel()
      this.scanCode()
    },
    // 扫码弹出取消
    scanCancel() {
      this.showScanTip = false
      this.showCancelButton = true
      // this.scanTipText=this.$$t("请先扫描二维码解锁设备！")
      this.scanConfirmText = this.$$t('立即扫码')
      this.scanCancelText = this.$$t('稍后处理')
    },
    // 录视频
    video() {
      callApi('MideaCommon', 'videoRecord', [5])
        .then(res => {
          console.log(res.video, '视频。。。。。。。。。。。。。')
          this.base(res.video, 'mp4')
        })
        .catch(err => {
          console.log(err)
        })
    },
    // 拍照
    photograph() {
      callApi('MCCamera', 'takePicture', [{ useGraffiti: 1, watermark: [`${uni.getStorageSync('userPermission').name}`, `${moment().format('YYYY-MM-DD HH:mm:ss')}`], sourceType: 1, size: 200 }])
        .then(res => {
          console.log(res.uri, '照片！！！！！！！！！')
          this.base(res.uri, 'png')
        })
        .catch(err => {
          console.log(err, '照片不行a')
        })
    },
    // 转base64
    base(url, type) {
      let _this = this
      this.checkNet()
        .then(() => {
          callApi('MideaCommon', 'getBase64s', [url])
            .then(res => {
              console.log(res.base64, '这是base64')
              let obj = {
                base64: res.base64[0],
                fileType: type,
                businessId: this.itemId,
                businessType: this.businessType,
                fileGroupId: this.fileGroupId
              }
              // 将数据发给后端
              uploadBase64(obj).then(res => {
                console.log(res, '发送回调数据')
                  //主设备上传图片以后，主设备防作弊设置为 false ,设备切换时不需要弹框
                  if(this.photoOrigin === 'dev'){
                    this.mainDevOptions[this.mainDevIndex].isNfcView = false;
                    this.list[this.mainDevIndex].isNfcView = false;
                    this.$set(this.list[this.mainDevIndex], 'forcedNfcCheck', '2')
        
                    console.info(this.list, 'forcedNfcCheckStatus')
                  }
                  //主设备上传图片以后，主设备防作弊设置为 false ,设备切换时不需要弹框
                  if(this.photoOrigin === 'dev'){
                    this.mainDevOptions[this.mainDevIndex].isNfcView = false;
                    this.list[this.mainDevIndex].isNfcView = false;
                    this.$set(this.list[this.mainDevIndex], 'forcedNfcCheck', '2')
        
                    console.info(this.list, 'forcedNfcCheckStatus')
                  }
                this.listbyfilegroupids()
              })
              // 调用转换文件流
            })
            .catch(err => {
              console.log(err, 'base64转换失败？')
            })
        })
        .catch(() => {
          callApi('MideaCommon', 'getBase64s', [url]).then(res => {
            console.log(res.base64, '这是base64')
            const annexName = `${tempSnowflake.nextId().toString()}.${type}`
            this.fileMsg.push({
              businessId: this.itemId,
              businessType: this.businessType,
              fileUrl: url,
              base64: res.base64,
              fileGroupId: this.fileGroupId,
              annexName, //离线的文件name由前端定义，
              annexFileType: type
            })
          })
        })
    },
    // 返回录音
    baseMp3() {
      this.listbyfilegroupids()
      // let obj = this.$store.state.mp3Path;
      // let checkPointItem = {};
      // this.list.forEach((v) => {
      //   v.secondaryDevList.forEach((vv) => {
      //     vv.checkPointList.forEach((vvv) => {
      //       if (vvv.id == this.itemId) {
      //         checkPointItem = vvv;
      //       }
      //     });
      //   });
      // });
      // if (
      //   checkPointItem.attachmentKey == null ||
      //   checkPointItem.attachmentKey == "null"
      // ) {
      //   checkPointItem.attachmentKey = [];
      // }
      // checkPointItem.attachmentKey.push(obj.id);
    },
    outLineMp3(url) {
      callApi('MideaCommon', 'getBase64s', [url]).then(res => {
        console.log(res, '这是base64')
        const annexFileType = url.slice(url.lastIndexOf('.') + 1)
        const annexName = `${tempSnowflake.nextId().toString()}.${annexFileType}`
        this.fileMsg.push({
          businessId: this.itemId,
          businessType: this.businessType,
          fileUrl: url,
          base64: res.base64,
          fileGroupId: this.fileGroupId,
          annexName, //离线的文件name由前端定义，
          annexFileType
        })
      })
    },
    // 用fileGroupId批量获取附件
    listbyfilegroupids() {
      this.checkNet().then(res => {
        let fileGroupIds = []
        //设备层级只有一个devFileGroupId， 测点层级需要循环测点取出fileGroupId
        if(this.photoOrigin === 'dev'){
          fileGroupIds.push(this.fileGroupId)
        }else{
          this.list.forEach(v => {
          v.secondaryDevList.forEach(vv => {
            vv.checkPointList.forEach(vvv => {
              fileGroupIds.push(vvv.fileGroupId)
            })
          })
        })
        }
        listbyfilegroupids({ fileGroupIds }).then(res => {
          if(this.photoOrigin === 'dev'){
            if(res.result.length > 0){
              //多张图片需要删除前一个
              if(res.result.length > 1){
                delAnnex({ ids: res.result[0].id }).then(res => {})
              }
              this.$set(this.list[this.mainDevIndex], 'fileGroup', [res.result[res.result.length -1]])
              this.devFilePath = `${window.globalConfig.VUE_APP_FILE_PREFIX}/eam/eambase/annex/preview?id=${res.result[res.result.length - 1].id}&token=${uni.getStorageSync("token")}`;
            }else{
              this.$set(this.list[this.mainDevIndex], 'fileGroup', [])
              this.devFilePath = ''
            }
            console.info(this.devFilePath, 'devFilePath')
          }else{
            this.list.forEach(v => {
              v.secondaryDevList.forEach(vv => {
                vv.checkPointList.forEach(vvv => {
                  let fileGroup = res.result.filter(v => v.fileGroupId === vvv.fileGroupId)
                  this.$set(vvv, 'fileGroup', fileGroup)
                })
              })
            })
          }
        })
      })
    },
    handleDevFileImgClose(){
      // 判断是否已提交，已提交数据不需要重置nfc标识
      if(this.mainDevOptions[this.mainDevIndex].uploadNum == 0){
        const originForcedNfcCheck = this.mainDevOptions[this.mainDevIndex].originForcedNfcCheck
        this.$set(this.list[this.mainDevIndex], 'forcedNfcCheck', originForcedNfcCheck)
        this.$set(this.mainDevOptions[this.mainDevIndex], 'isNfcView', originForcedNfcCheck == 1 ? true : false)
        this.$set(this.list[this.mainDevIndex], 'isNfcView', originForcedNfcCheck == 1 ? true : false)
      }
      delAnnex({ ids: this.list[this.mainDevIndex].fileGroup[0].id }).then(res => {
        this.listbyfilegroupids()
      })
    },
    getFullName(text, type) {
      const strArr = text.split('/')
      if (type === 1) {
        let str = ''
        for (var i = 0; i < strArr.length - 1; i++) {
          str = str + (i === 0 ? '' : '/') + strArr[i]
        }
        return str + '/'
      } else {
        return strArr[strArr.length - 1]
      }
    },
    showAlarmSelect(checkPointIndex, lv, index1, index2, dom) {
      this.alarmSelectValue = lv
      this.index1 = index1
      this.index2 = index2
      this.index3 = checkPointIndex
      // let html = document.getElementById('alarmSelect'+checkPointIndex).getBoundingClientRect()
      const html = dom.target
      if (html.y > 400) {
        this.alarmTop = html.y - 225
        this.alarmLeft = html.x - 90
      } else {
        this.alarmTop = html.y + 25
        this.alarmLeft = html.x - 90
      }
      this.popupShow = true
      this.alarmLevelShow = !this.alarmLevelShow
    },
    toShowGuideMap(type, item = {}) {
      console.log(item, 'lllll')
      this.checkPointImg = item
      if (type) {
        if (item.pointType !== '0') {
          this.popShow(item)
          this.current = 0
        } else {
          this.current = 1
        }
        if (item.pointFileGroupId) {
          listbyfilegroupid({ fileGroupId: item.pointFileGroupId }).then(res => {
            this.fileStandardList = res.result.map(item => {
              item.fileSrc = window.globalConfig.VUE_APP_FILE_PREFIX + item.annexUrl
              return item
            })
            this.$refs.standardPopup.open('center')
          })
        } else {
          this.fileStandardList = []
          this.$refs.standardPopup.open('center')
        }
      } else {
        this.$refs.standardPopup.close()
      }
    }
  }
}
</script>

<style scoped lang="less">
.searchBtn {
  margin-right: 15rpx;
}

.keyIns {
  height: calc(100vh - 100rpx) !important;
}
.inspectEdit {
  background: #f5f7fa;
  // min-height: 100vh;
  height: calc(100vh - 220rpx);

  // margin-bottom: 132rpx;
  .head {
    height: 336rpx;
  }

  .nodeName {
    width: 100%;
    line-height: 72rpx;
    padding: 0 32rpx;
    font-size: 24rpx;
    color: #909399;
    background: #fff;
    background: #f5f7fa;
    // overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    box-sizing: border-box;

    span {
      color: #3b71e8;
    }
  }

  .main {
    position: relative;
  }

  .main.inspectTitle {
    width: 100%;
    box-sizing: border-box;
  }

  .inspectTitle {
    height: 132rpx;
    font-size: 32rpx;
    color: #303133;
    padding: 0 32rpx;
    display: flex;
    flex-direction: column;
    background: #fff;
    .devType {
      padding: 24rpx 0 24rpx 0;
      font-family: PingFangSC-Regular;
      font-weight: 400;
      font-size: 20rpx;
      color: #909399;
      line-height: 20rpx;
      .img{
        width: 28rpx;
        height:28rpx;
        float: right;
      }
    }
    .flex_left_box,
    .partEquipment {
      font-size: 28rpx;
      color: #303133;
    }

    .select {
      flex-shrink: 0;
      margin-left: 8rpx;
      padding: 4rpx 12rpx;
      display: flex;
      align-items: center;
      border-radius: 4rpx;
      font-size: 20rpx;

      .state {
        display: inline-block;
        width: 8rpx;
        height: 8rpx;
        border-radius: 8rpx;
        margin-right: 10rpx;
      }

      &.one {
        background-color: rgba(#2da641, 0.1);
        span {
          &.state {
            background-color: #2da641;
          }
          color: #2da641;
        }
      }

      &.two {
        background-color: rgba(#ed6a0c, 0.1);
        span {
          &.state {
            background-color: #ed6a0c;
          }
          color: #ed6a0c;
        }
      }

      &.three {
        background-color: rgba(#dbae02, 0.2);
        span {
          &.state {
            background-color: #dbae02;
          }
          color: #dbae02;
        }
      }

      &.four {
        background-color: rgba(#d40000, 0.1);
        span {
          &.state {
            background-color: #d40000;
          }
          color: #d40000;
        }
      }
      &.five {
        background-color: rgba(#4964e1, 0.1);
        span {
          &.state {
            background-color: rgba(#4964e1, 0.8);
          }
          color: rgba(#4964e1, 0.8);
        }
      }
    }
    .mainEquipmentNum {
      font-family: PingFangSC-Regular;
      font-weight: 400;
      font-size: 28rpx;
      text-align: right;
      line-height: 28rpx;
    }
    .partEquipment {
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      display: inline-block;
      width: 640rpx;
    }
  }

  .part-dev {
    width: 100%;
    box-sizing: border-box;
  }
  .inspect-list {
    height: calc(100vh - 560rpx);
    height: calc(100vh - 560rpx - constant(safe-area-inset-bottom));
    height: calc(100vh - 560rpx - env(safe-area-inset-bottom));
    overflow-y: auto;
    overflow-x: hidden;
    .inspectDetail {
      margin-top: 16rpx;
      background: #fff;
      padding: 32rpx;

      .title {
        font-size: 32rpx;
        color: #303133;
        font-weight: 600;
        display: flex;
        flex-direction: row;
        align-items: center;

        .secondaryDevName {
          display: inline-block;
          width: 612rpx;
          word-break: break-all;
        }

        .modify {
          font-size: 28rpx;
          color: #3b71e8;
        }
      }

      .name {
        font-weight: 400;
        color: #303133;
        font-size: 32rpx;
        margin-top: 32rpx;
        padding-right: 50rpx;
        display: flex;
        flex-direction: row;
        align-items: center;
        position: relative;

        .left {
          // background: #3B71E8;
          // transform: rotate(45deg);
          // display: inline-block;
          height: 60rpx;
          display: grid;
          width: 60rpx;
          margin-right: 10rpx;
        }

        .iconZero {
          height: 100%;
          width: 100%;
          background-repeat: no-repeat;
          background-image: url(../../static/img/observation.svg);
        }

        .iconOne {
          height: 100%;
          width: 100%;
          background-repeat: no-repeat;
          background-image: url(../../static/img/speed.svg);
        }

        .iconTwo {
          // width: 44rpx;
          // height: 46rpx;
          height: 100%;
          width: 100%;
          background-repeat: no-repeat;
          background-image: url(../../static/img/temperature.svg);
        }

        .iconThree {
          height: 100%;
          width: 100%;
          background-repeat: no-repeat;
          background-image: url(../../static/img/workmanship.svg);
        }

        .iconFour {
          height: 100%;
          width: 100%;
          background-repeat: no-repeat;
          background-image: url(../../static/img/switch.svg);
        }

        .right {
          position: absolute;
          right: 0;
          top: 0;
          width: 48rpx;
          height: 28rpx;
          background: #f0f2f5;
          border-radius: 4rpx;
          font-size: 20rpx;
          color: #999;
        }
      }
      .content-warp {
        margin-top: 24rpx;
        background: #f5f7fa;
        border-radius: 8rpx;
        padding: 0 24rpx;
        > .content {
          display: flex;
          flex-direction: row;
          font-family: PingFangSC-Regular;
          font-weight: 400;
          font-size: 24rpx;
          color: #606266;
          line-height: 24rpx;
          padding-top: 24rpx;
        }
      }

      .singleGroup.content {
        width: 100%;
        height: 88rpx;
        display: flex;
        justify-content: flex-start;
      }

      .chooses {
        margin-bottom: 28rpx;
        .tip {
          font-family: PingFangSC-Regular;
          font-weight: 400;
          font-size: 24rpx;
          color: #606266;
          line-height: 24rpx;
          margin-bottom: 16rpx;
        }
        .checkbox {
          display: flex;
          flex-wrap: wrap;
          button {
            flex-shrink: 0;
            padding: 16rpx 32rpx;
            min-width: 180rpx;
            box-sizing: border-box;
            background: rgba(59, 113, 232, 0.1);
            border-radius: 4rpx;
            font-family: PingFangSC-Regular;
            font-weight: 400;
            font-size: 20rpx;
            color: #3b71e8;
            margin: 0 8rpx 8rpx 0;
            border: 1rpx solid #3b71e8;
            word-break: break-all;
            max-width: 100%;
            &.isChoices {
              background: #3b71e8;
              color: #ffffff;
              border-width: 0;
            }
          }
        }
      }

      .inputBox {
        width: 640rpx;
        height: 80rpx;
        padding: 20rpx 0;

        .unit {
          float: right;
          margin-top: 10rpx;
          font-size: 36rpx;
          font-weight: 100;
        }
      }

      .inputBox /deep/ .u-input {
        width: 480rpx;
        float: left;
        font-size: 14px;
      }
    }
  }
  .keyUp {
    height: calc(100vh - 300rpx) !important;
    height: calc(100vh - 300rpx - constant(safe-area-inset-bottom)) !important;
    height: calc(100vh - 300rpx - env(safe-area-inset-bottom)) !important;
  }
}
.enclosure-warp {
  margin-top: 24rpx;
  background: #f5f7fa;
  border-radius: 8rpx;
  padding: 0 24rpx;
  .enclosure-title {
    font-family: PingFangSC-Regular;
    font-weight: 400;
    font-size: 24rpx;
    color: #606266;
    line-height: 24rpx;
    padding-top: 24rpx;
    padding-top: 24rpx;
    text {
      color: #606266;
      font-weight: 500;
    }
  }
  .content {
    > view {
      padding: 24rpx 0;
      border-bottom: 1px solid #e4e7ed;
      &:last-of-type {
        border-width: 0;
      }
      .left {
        width: calc(100% - 60rpx);
        .icon {
          width: 64rpx;
          height: 64rpx;
          margin-right: 16rpx;
          image {
            width: 64rpx;
            height: 64rpx;
          }
        }
        .info {
          width: 100px;
          flex: 1;
          height: 64rpx;
          flex-direction: column;
          align-items: flex-start;
          view {
            &:nth-of-type(1) {
              width: 100%;
              font-family: PingFangSC-Regular;
              font-size: 28rpx;
              color: #303133;
              line-height: inherit;
            }
            &:nth-of-type(2) {
              font-family: PingFangSC-Medium;
              font-weight: 500;
              font-size: 24rpx;
              color: #909399;
              letter-spacing: 0;
              line-height: 24rpx;
              text {
                color: #19c361;
                &.notUploaded {
                  color: #e94141;
                }
              }
            }
          }
        }
      }
      .right {
        flex-shrink: 0;
        width: 50rpx;
        height: 50rpx;
        background: #e1e6ee;
        border-radius: 100%;
        image {
          width: 50rpx;
          height: 50rpx;
        }
      }
    }
  }
}
.upAndDownBtn {
  position: fixed;
  left: 0;
  bottom: 0rpx;
  width: calc(100% - 64rpx);
  height: 132rpx;
  background-color: white;
  border-top: 2rpx solid #f5f7fa;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 32rpx;
  padding-bottom: 0;
  padding-bottom: constant(safe-area-inset-bottom);
  padding-bottom: env(safe-area-inset-bottom);
  button {
    display: flex;
    width: 90rpx;
    height: 92rpx;
    background: #ffffff;
    border: 1px solid #3b71e8;
    border-radius: 8rpx;
    padding: 0 !important;
    &[disabled] {
      border: 1px solid rgba(#2d79fd, 0.8);
    }
  }
  .step {
    &:first-of-type {
      margin-right: 20rpx;
    }
    &:last-of-type {
      margin-left: 20rpx;
    }
    &[disabled] {
      border-color: #ccc;
      color: #fff;
    }
  }
  .upload {
    flex: 1;
    background-color: #3b71e8;
    border-radius: 0 8rpx 8rpx 0;
    &.singleDevice {
      border-radius: 8rpx;
      border-left: 1px;
    }
    &[disabled] {
      background-color: rgba(#3b71e8, 0.8);
      color: #fff;
    }
  }
  .upload.batchUpload {
    border-radius: 8rpx 0 0 8rpx;
    flex: 1;
    flex-basis: auto;
    background-color: #fff;
    color: #3b71e8;
    display: flex;
    &[disabled] {
      background-color: rgba(#fff, 0.8);
      color: rgba(#2d79fd, 0.8);
    }
  }
}

ul {
  position: fixed;
  top: 280rpx;
  // right: 20rpx;
  left: 32rpx;
  // width: 155rpx;
  // height: 530rpx;
  z-index: 10072;
  border-radius: 10rpx;
  background-color: #ffffff;
  box-shadow: 0px 1px 2px 1px rgba(#333, 0.2);
}

ul li:nth-child(1) {
  color: #2da641;
  height: 104rpx;
  line-height: 104rpx;
  border-bottom: 1px solid #f5f7fa;
  // list-style-type: disc;
}

ul li:nth-child(2) {
  color: #ed6a0c;
  height: 104rpx;
  line-height: 104rpx;
  border-bottom: 1px solid #f5f7fa;
}

ul li:nth-child(3) {
  color: #dbae02;
  height: 104rpx;
  line-height: 104rpx;
  border-bottom: 1px solid #f5f7fa;
}

ul li:nth-child(4) {
  color: #d40000;
  height: 104rpx;
  line-height: 104rpx;
  border-bottom: 1px solid #f5f7fa;
}

ul li:nth-child(5) {
  color: rgba(#4964e1, 0.8);
  height: 104rpx;
  line-height: 104rpx;
}

ul li span {
  margin-right: 20rpx;
  color: black;
}

.lock {
  width: 20rpx;
}

#problem {
  width: 20rpx;
  height: 20rpx;
}

.equipmentShow /deep/ .u-icon--right {
  display: inline-block;
}

.overhaulPop /deep/ .uni-popup-dialog {
  width: 570rpx;
  height: 376rpx;
}

.overhaulPop /deep/ .uni-dialog-content {
  flex-direction: column;
  height: 282rpx;
  padding: 0;
}

.overhaulPop.confirmPopup {
  .confirmPopup-content {
    border-radius: 48rpx;
    width: 686rpx;
    height: auto;
    max-height: 72vh;
    box-sizing: border-box;
    padding: 0 40rpx 40rpx 40rpx;
    flex: 1;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    background-color: #ffffff;
    left: 32rpx;

    .scroll-view {
      flex: 1;
      display: flex;
      overflow: hidden;

      scroll-view {
        flex: 1;
      }

      .select {
        padding: 4rpx 12rpx;
        display: flex;
        align-items: center;
        border-radius: 4rpx;
        font-size: 20rpx;

        &.one {
          background-color: rgba(#2da641, 0.1);
          color: #2da641;
        }

        &.two {
          background-color: rgba(#ed6a0c, 0.1);
          color: #ed6a0c;
        }

        &.three {
          background-color: rgba(#dbae02, 0.2);
          color: #dbae02;
        }

        &.four {
          background-color: rgba(#d40000, 0.1);
          color: #d40000;
        }
        &.five {
          background-color: rgba(#4964e1, 0.1);
          color: rgba(#4964e1, 0.8);
        }
      }
    }
    .btns {
      button {
        width: 252rpx;
        height: 92rpx;
        background: #f0f2f5;
        border-radius: 8rpx;
        font-family: PingFangSC-Regular;
        font-weight: 400;
        font-size: 28rpx;
        color: #909399;
        padding: 24rpx 0;
      }
      .confirm {
        width: 336rpx !important;
        background: #3b71e8;
        width: 112px;
        color: #ffffff;
      }
    }
  }
}

.confirmPopup {
  .title {
    padding-top: 40rpx;
    font-family: PingFangSC-SNaNpxibold;
    font-weight: 600;
    font-size: 32rpx;
    color: #303133;
    line-height: 32rpx;
  }
  .title_sub {
    padding: 16rpx 0 32rpx 0;
    font-family: PingFangSC-Regular;
    font-weight: 400;
    font-size: 24rpx;
    color: #606266;
    line-height: 28rpx;
  }
  .secondaryDev {
    width: 606rpx;
    background: #f5f7fa;
    border-radius: 8px;
    padding: 0 32rpx 1rpx 32rpx;
    margin-bottom: 24rpx;
    box-sizing: border-box;
    word-break: break-all;

    .secondaryDevName {
      font-family: PingFangSC-Medium;
      line-height: 60rpx;
      font-weight: 500;
      font-size: 28rpx;
      color: #303133;
      line-height: 1;
      padding: 24rpx 0 32rpx 0;
    }
    .point {
      margin-bottom: 24rpx;
      // align-items: flex-start;
      .pointName {
        font-family: PingFangSC-Regular;
        font-weight: 400;
        font-size: 24rpx;
        color: #606266;
        line-height: 1;
        flex-shrink: 0;
        max-width: 70%;
      }

      .singleChoice {
        width: 100px;
        flex: 1;
        margin-left: 20rpx;
        display: flex;
        justify-content: center;
        align-items: flex-end;
        flex-direction: column;
        font-weight: 400;
        font-size: 24rpx;
        .pointState {
        }
        .errorOption {
          view {
            margin-left: 20rpx;
          }
        }
        // .other-option{
        // 	width: 100%;
        // }
      }

      .inputBox {
        font-size: 28rpx;
        color: #999;
        word-break: break-all;
        flex-shrink: 0;
      }
    }
  }
}

.overhaulPop.successPopup /deep/ .uni-dialog-button:nth-of-type(1) {
  display: none;
}

.correct_img {
  width: 107.5rpx;
  height: 107.5rpx;
  margin-bottom: 24rpx;
}
.preview-warp {
  .correct_img {
    width: 500rpx;
    height: 500rpx;
  }
  .correct_amr {
    width: 300rpx;
  }
}
/deep/ .uni-audio-default {
  min-width: 300rpx;
}
// .correct_title {

// }
.chooseGreen /deep/ .u-radio__icon-wrap--circle {
  background-color: #2da641 !important;
  border-color: #2da641 !important;
}

.chooseGreen /deep/ .u-radio__text {
  color: #2da641 !important;
}

.chooseOrange /deep/ .u-radio__icon-wrap--circle {
  background-color: #ed6a0c !important;
  border-color: #ed6a0c !important;
}

.chooseOrange /deep/ .u-radio__text {
  color: #ed6a0c !important;
}

.unEdit_show {
  margin-left: 80rpx;
  font-size: 14rpx;

  .unEdit_sign {
    display: inline-block;
    width: 8rpx;
    height: 8rpx;
    border-radius: 4rpx;
    margin: 0 8rpx 5rpx 0;
    background: rgba(237, 106, 12, 0.8);
  }

  .unEdit_word {
    font-size: 28rpx;
  }
}

.popCloseBox {
  position: fixed;
  top: 0;
  width: 750rpx;
  height: 2000rpx;
  // width: 100%;
  // height: 500rpx;
  // width: 500rpx;
  // background-color: gray;
  // opacity: 0.3;
  z-index: 10071;
}

/deep/ .u-modal__content__text span {
  display: block;
  text-align: center;
}

.other-option {
  position: relative;
  margin-top: 20upx;

  .alarm-lv {
    font-size: 28upx;
    // position: absolute;
    // right: 0;
    // top: -60upx
  }
  .tip {
    font-family: PingFangSC-Regular;
    font-weight: 400;
    font-size: 24rpx;
    color: #606266;
    line-height: 24rpx;
  }
  /deep/ .u-textarea {
    background-color: transparent;
    border-width: 0 !important;
    padding: 0 !important;
    .uni-textarea-textarea {
      font-size: 28rpx;
    }
  }
}

.alarm-select {
  position: fixed;
  // right: 20upx;
  // top: 100upx;
  // width: 300rpx;
  height: 424rpx;
  z-index: 10072;
  right: 10px;
  border-radius: 10rpx;
  background-color: #ffffff;
  box-shadow: 0px 1px 2px #3c9cff80;

  .alarm-item:nth-child(1) {
    color: #2da641;
    height: 104rpx;
    line-height: 104rpx;
    border-bottom: 1px solid #f5f7fa;
    // list-style-type: disc;
  }

  .alarm-item:nth-child(2) {
    color: #ed6a0c;
    height: 104rpx;
    line-height: 104rpx;
    border-bottom: 1px solid #f5f7fa;
  }

  .alarm-item:nth-child(3) {
    color: #dbae02;
    height: 104rpx;
    line-height: 104rpx;
    border-bottom: 1px solid #f5f7fa;
  }

  .alarm-item:nth-child(4) {
    color: #d40000;
    height: 104rpx;
    line-height: 104rpx;
  }

  .alarm-item span {
    margin: 0 20rpx;
    color: black;
  }

  .alarm-item .icon {
    width: 16upx;
    height: 16upx;
    border-radius: 50%;
    margin-left: 20upx;
  }

  .alarm-item {
    display: flex;
    align-items: center;
  }
}
.enclosureAction {
  position: fixed;
  bottom: 0;
  width: 686rpx;
  height: 524rpx;
  background: #ffffff;
  border-radius: 48rpx;
  left: 32rpx;
  bottom: calc(76rpx + constant(safe-area-inset-bottom));
  bottom: calc(76rpx + env(safe-area-inset-bottom));
  padding: 0 40rpx;
  box-sizing: border-box;
  .title {
    padding-top: 40rpx;
    font-family: PingFangSC-SNaNpxibold;
    font-weight: 600;
    font-size: 32rpx;
    color: #303133;
    line-height: 32rpx;
  }
  .title_sub {
    padding: 16rpx 0 24rpx 0;
    font-family: PingFangSC-Regular;
    font-weight: 400;
    font-size: 24rpx;
    color: #303133;
    line-height: 28rpx;
  }
  .fileList {
    width: 100%;
    .fileList-item {
      width: 190rpx;
      height: 144rpx;
      background: #f0f2f5;
      border-radius: 8rx;
      font-family: PingFangSC-Regular;
      font-weight: 400;
      font-size: 24rpx;
      color: #303133;
      line-height: 24rpx;
      image {
        width: 46rpx;
        height: 46rpx;
        margin-bottom: 16rpx;
      }
    }
  }
  .enclosure_remark {
    height: 92rpx;
    background: #f0f2f5;
    border-radius: 8rpx;
    font-family: PingFangSC-Regular;
    font-weight: 400;
    font-size: 24rpx;
    color: #303133;
    padding: 0 32rpx;
    margin: 24rpx 0;
  }
  .btn {
    font-family: PingFangSC-Regular;
    font-weight: 400;
    font-size: 28rpx;
    color: #e94141;
    padding: 24rpx 0;
  }
}
.standardPopup {
  width: 90vw;
  height: 50vh;
  background-color: white;
  display: flex;
  flex-direction: column;
  align-items: center;
  border-radius: 10rpx;
  overflow: hidden;
  swiper,
  swiper-item {
    width: 100%;
    height: 100%;
  }
  /deep/.u-mode-center-box {
    background-color: rgba(0, 0, 0, 0) !important;
  }
  image {
    width: 100%;
    height: 90%;
  }
  &_title {
    //padding: 20rpx 0;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 100rpx;
    > view {
      width: 50%;
      height: 100%;
      line-height: 100rpx;
      text-align: center;
    }
    .active {
      background-color: #3b71e8;
      color: white;
    }
  }
  .slot-content {
    height: 40vh;
    padding: 20rpx;
    box-sizing: border-box;
    line-height: 80rpx;
  }
  &_img {
    width: 90%;
    height: 36vh;
    max-height: 36vh;
    margin: 20rpx 30rpx;
    box-sizing: border-box;
    border-bottom: 1px solid #e5e5e5;
  }
  &_btn {
    color: #007aff;
    padding-bottom: 20rpx;
    width: 100%;
    text-align: center;
  }
}
.devFilePopup {
  width: 90vw;
  min-height: 20vh;
  background-color: white;
  display: flex;
  flex-direction: column;
  align-items: center;
  border-radius: 10rpx;
  overflow: hidden;
  swiper,
  swiper-item {
    width: 100%;
    height: 100%;
  }
  /deep/.u-mode-center-box {
    background-color: rgba(0, 0, 0, 0) !important;
  }
  image {
    width: 100%;
    height: 90%;
  }
  &_title {
    //padding: 20rpx 0;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 100rpx;
    > view {
      width: 50%;
      height: 100%;
      line-height: 100rpx;
      text-align: center;
    }
    .active {
      background-color: #3b71e8;
      color: white;
    }
  }
  &_img{
    font-size: 26rpx;
    .img{
      width: 50rpx;
      height: 50rpx;
    }
  }
  &_preview{
    height:52vh;
    width: calc(100vw - 100rpx);
    margin: 20rpx 0;
    position: relative;
    text-align: center;
    margin-bottom: -60rpx;
    .close{
      position: absolute;
      right:10rpx;
      top:-40rpx;;
      width: 60rpx;
      height: 60rpx;
      z-index: 1;
    }
    .img{
      width: 100% !important;
      // height: 540rpx;
    }
  }
  &_btn {
    color: #007aff;
    padding-bottom: 20rpx;
    width: 100%;
    text-align: center;
    overflow: hidden;
    margin-top: 30rpx;
    position: relative;
    z-index: 10;
  }
  .noData{
    text-align: center;
    color: #999;
    margin: 40rpx 0;
  }
}
.no_data {
  position: fixed;
  margin: auto;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  width: 80%;
  height: 90%;
  image {
    height: 20vh;
  }
  .no_data_title {
    margin: 30rpx auto;
    // width: 220rpx;
    color: #babfc9;
    font-size: 40rpx;
    text-align: center;
  }
}

</style>
