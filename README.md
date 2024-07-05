incontactAgentConfigPage.vfp

<apex:page tabStyle="icAgentConsole__incAgentConfiguration__c" controller="icAgentConsole.inContactAgentConfigController" action="{!pageInit}" standardStylesheets="false">
  <script type="text/javascript" src="{!URLFOR($Resource.AgentConsoleApp, 'js/lib/jquery-3.5.0.min.js')}"/>
  <script type="text/javascript" src="{!URLFOR($Resource.AgentConsoleApp, 'js/lib/log4javascript.js')}"/>
  <script type="text/javascript" src="{!URLFOR($Resource.AgentConsoleApp, 'js/common/SessionStorageAppender.js')}"/>
  <script type="text/javascript" src="{!URLFOR($Resource.AgentConsoleApp, 'js/common/Util.js')}"/>
  <script type="text/javascript" src="{!URLFOR($Resource.AgentConsoleApp, 'js/common/AjaxHelper.js')}"/>
  <script type="text/javascript" src="{!URLFOR($Resource.AgentConsoleApp, 'js/data/AuthenticationDataStore.js')}"/>
  
  <script src="/soap/ajax/52.0/connection.js" type="text/javascript"/>

  <apex:form styleClass="icAgentconsole">
    <apex:inputText style="display:none;" styleClass="_sfMode" value="{!sfMode}" id="_sfMode" />
    <apex:actionStatus id="updateProfileStatus">
      <apex:facet name="start">
        <div class="slds-spinner_container">
          <div class="slds-spinner--brand slds-spinner slds-spinner--large" role="alert">
            <span class="slds-assistive-text"></span>
            <div class="slds-spinner__dot-a"></div>
            <div class="slds-spinner__dot-b"></div>
          </div>
        </div>
        <div>
          <div class="loading-layer" />
          <div class="loading-panel">
            <span class="loading-icn"></span>
            <span class="loading-txt">{!$Label.processing}...</span>
          </div>
        </div>
      </apex:facet>
      <apex:facet name="stop">
        <apex:outputPanel layout="block" rendered="true" id="masterPanel" styleClass="masterPanelStyle">
          <div class="header">
            <div class="floating">
              <div class="title-panel">
                <div class="slds-icon_container setting-icn icn" title="{!$Label.salesforceAgent}">
                  <img src="{!URLFOR($Resource.AgentConsoleApp,'css/images/servlet.png')}" width="38px" />
                </div>
                <div class="title">
                  <div class="primary">
                    <label>{!$Label.agentConsoleConfiguration}</label>
                  </div>
                  <div class="secondary">
                    <label>{!$Label.configurationSettings}</label>
                  </div>
                </div>
                <div class="buttons config-action">
                  <apex:commandButton title="{!$Label.icagentconsole__cancel}" value="{!$Label.icagentconsole__cancel}" action="{!cancelConfiguration}" reRender="parentOutputPanel, messages, scriptPanel"
                    styleClass="slds-button slds-button--neutral btn" id="btn_cancel" />
                  <input type="button" value="{!$Label.save}" class="slds-button slds-button--brand btn" onclick="saveConfiguration();" id="btn_save"
                  />
                </div>
              </div>
            </div>
          </div>

          <apex:outputPanel styleClass="message-panel" id="messages">
            <div class="msg-container {!IF(pageMessage.messageType != '','show-container','')} {!LOWER(pageMessage.messageType)}">
              <h2 style="padding: 8px;">
                <apex:image width="28" rendered="{!IF(AND(pageMessage.messageType != '', IsLightningMode) , true, false)}" url="{!URLFOR($Resource.icAgentConsole__AgentConsoleApp,'css/images/lightning/'+LOWER(pageMessage.messageType)+'.png')}"
                />
                <apex:image width="28" rendered="{!IF(AND(pageMessage.messageType != '', $User.UITheme == 'Theme3') , true, false)}" url="{!URLFOR($Resource.icAgentConsole__AgentConsoleApp,'css/images/'+LOWER(pageMessage.messageType)+'.png')}"
                  styleClass="alertIconClassic" />
                <span class="messageTxt">{!pageMessage.message}</span>
                <apex:outputPanel styleClass="close_icn" onclick="closeAlert();" rendered="{!IF(pageMessage.messageType != '' , true, false)}">
                  &#10006;
                </apex:outputPanel>
              </h2>
            </div>
          </apex:outputPanel>

          <div class="content bPageBlock brandSecondaryBrd apexDefaultPageBlock secondaryPalette remove-border">
            <table style="width:100%;">
              <tr>
                <td style="width:20%; vertical-align:100%;horizontal-align:left;">
                  <span style="font-weight: bold;font-size: 14px;margin-left: 1px;">{!$Label.profiles}</span>
                  <br />
                  <br />
                  <div class="slds-form-element__row">
                    <div class="slds-form-element">
                      <div class="slds-form-element__control">
                        <div class="slds-select_container">
                          <apex:selectlist id="ddlProfileNames" style="vertical-align:100%;" multiselect="false" styleClass="slds-select" onchange="closeAlert();actionProfileSelected()"
                            value="{!selectedProfileId}" size="{!IF(sfMode == 'Lightning', 12, 20)}">
                            <apex:selectOptions value="{!profileNames}"></apex:selectOptions>
                          </apex:selectlist>
                        </div>
                      </div>
                    </div>
                  </div>
                  <apex:outputPanel id="cxoneRoutingPanel">
                    <div class="cxone-routing border-space bPageBlock" style="display:{!IF(isOmniChannelEnabled == true ,'block','none')}">
                      <span style="font-weight: bold;font-size: 14px;margin-left: 1px;">{!$Label.configOrgWideSettings}</span>
                      <br />
                      <br />
                      <div class="slds-form-element__row">
                        <div class="slds-form-element">
                          <div class="slds-form-element__control">
                            <label class="slds-checkbox" for="enableCxOneRouting">
                              <input name="enableCxOneRouting" type="checkbox" id="enableCxOneRouting" onchange="onToggleCxOneRouting(this.checked)" value="{!sfOmniRoutingSettings.isEnabled}"
                              />
                              <span class="slds-checkbox--faux"></span>
                              <span class="slds-form-element__label enable-cxone-routing">{!$Label.enableCxOneRoutingLabel}</span>
                            </label>
                          </div>
                        </div>
                      </div>
                      <div class="slds-form-element__row">
                        <div class="slds-form-element">
                          <label class="slds-form-element__label" for="sfUsername">{!$Label.salesforceUsername}</label>
                          <div class="slds-form-element__control">
                            <input id="sfUsername" placeholder="{!$Label.username}" class="slds-input" type="text" value="{!sfOmniRoutingSettings.salesforceUsername}"
                              disabled="true" />
                          </div>
                        </div>
                      </div>
                      <div class="slds-form-element__row">
                        <div class="slds-form-element">
                          <label class="slds-form-element__label" for="sfPassword">{!$Label.salesforcePassword}</label>
                          <div class="slds-form-element__control">
                            <input id="sfPassword" placeholder="{!$Label.password}" class="slds-input" type="password" value="{!sfOmniRoutingSettings.salesforcePassword}"
                              disabled="true" />
                            <div id="togglePwdIcon" class="toggle-pwd-icon" onclick="onTogglePwdIcon()"></div>
                          </div>
                        </div>
                      </div>
                      <div class="slds-form-element__row">
                        <div class="slds-form-element">
                          <label class="slds-form-element__label" for="pushTopicName">{!$Label.salesforcePushTopicName}</label>
                          <div class="slds-form-element__control">
                            <input id="pushTopicName" class="slds-input" type="text" value="{!sfOmniRoutingSettings.salesforcePushTopic}" disabled="true"
                            />
                          </div>
                        </div>
                      </div>
                      <div class="slds-form-element__row">
                        <div class="slds-form-element">
                          <label class="slds-form-element__label" for="pocId">{!$Label.pointOfContactId}</label>
                          <div class="slds-form-element__control">
                            <input id="pocId" class="slds-input" type="number" value="{!sfOmniRoutingSettings.pointOfContactId}" disabled="true" />
                          </div>
                        </div>
                      </div>
                    </div>
                    <script type="text/javascript">
                      function canSaveConfiguration() {
                        var isCxoneRoutingValid = {!isCxoneRoutingValid                      
};      
                        triggerSaveConfiguration();
                      setCxOneRouting();
                      }
                    </script>
                  </apex:outputPanel>
                </td>
                <td style="width:80%; vertical-align:top;horizontal-align:left;padding-left:20px;">
                  <apex:outputPanel id="parentOutputPanel">
                    <apex:outputText style="font-size: 14px;margin-left: 1px;">
                      {!$Label.configure}
                      <b>{!SelectedProfileName}</b> {!$Label.profile}
                    </apex:outputText>
                    <apex:panelGroup id="profileOptionsGroup" rendered="{!selectedProfileId != '_default'}">
                      <div class="slds-form-element">
                        <div class="slds-form-element__control">
                          <label class="slds-radio" for="optionDefault" style="float: left;">
                            <input type="radio" name="profileOptions" id="optionDefault" value="_default" />
                            <span class="slds-radio--faux"></span>
                            <span class="slds-form-element__label">{!$Label.useDefault}</span>
                          </label>
                          <label class="slds-radio" for="optionCustom">
                            <input type="radio" name="profileOptions" id="optionCustom" value="custom" />
                            <span class="slds-radio--faux"></span>
                            <span class="slds-form-element__label">{!$Label.custom}</span>
                          </label>
                        </div>
                      </div>
                    </apex:panelGroup>
                    <apex:actionStatus id="customSettingStatus">
                      <apex:facet name="start">
                        <div class="slds-spinner_container">
                          <div class="slds-spinner--brand slds-spinner slds-spinner--large" role="alert">
                            <span class="slds-assistive-text"></span>
                            <div class="slds-spinner__dot-a"></div>
                            <div class="slds-spinner__dot-b"></div>
                          </div>
                        </div>
                        <div>
                          <div class="loading-layer" />
                          <div class="loading-panel">
                            <span class="loading-icn"></span>
                            <span class="loading-txt">{!$Label.processing}...</span>
                          </div>
                        </div>
                      </apex:facet>
                      <apex:facet name="stop">
                        <apex:outputPanel id="cutomProfileOptionPanel">
                          <apex:outputpanel rendered="{!selectedProfileOption == 'custom'}">
                            <table style="width:100%;">
                              <!--ST-->
                              <tr>
                                <!-- display table-cell to span the td for 2 columns -->
                                <td style="width:100%;display:{!IF(isOmniChannelEnabled == true ,'table-cell','none')}" colspan="2">
                                  <div style="padding-top: 10px;">
                                    <span style="font-weight: bold;">{!$Label.presenceSyncSettings}</span>
                                    <div style="padding-top: 4px;padding-bottom: 5px;">
                                      <div class="slds-form-element__row">
                                        <div class="slds-form-element">
                                          <div class="slds-form-element__control">
                                            <label class="slds-checkbox" for="chkAgentAppSync">
                                              <input name="default" type="checkbox" id="chkAgentAppSync" value="{!isOmnichannelPresenceSyncEnabled}" onchange="resetAgentSyncSelectionJS(this.checked);"
                                              />
                                              <span class="slds-checkbox--faux"></span>
                                              <span class="slds-form-element__label">{!$Label.enableAgentPresenceSync}</span>
                                            </label>
                                          </div>
                                        </div>
                                      </div>
                                      <div style="margin-top: 10px;display:{!IF(isOmnichannelPresenceSyncEnabled == true ,'block','none')}">
                                        <div class="slds-form-element" style="margin-bottom: 5px;">
                                          <span style="font-weight: bold;">{!$Label.presenceMaster}</span>
                                          <div class="slds-form-element__control" style="margin-top: 5px;">
                                            <label class="slds-radio" for="inContact" style="float: left;margin-right: 20px;">
                                              <input type="radio" name="isMasterApp" id="inContact" value="{!$Label.inContact}" />
                                              <span class="slds-radio--faux"></span>
                                              <span class="slds-form-element__label">{!$Label.inContact}</span>
                                            </label>
                                            <label class="slds-radio" for="salesforce">
                                              <input type="radio" name="isMasterApp" id="salesforce" value="{!$Label.salesforce}" />
                                              <span class="slds-radio--faux"></span>
                                              <span class="slds-form-element__label">{!$Label.salesforce}</span>
                                            </label>
                                          </div>
                                        </div>

                                        <span style="font-weight: bold;padding-top: 1px;">{!$Label.stateMappings}</span>
                                        <span class="addPresenceMappingBtn">
                                          <apex:commandButton id="addPresenceMappingBtn" value="{!$Label.icagentconsole__addpresencemapping}" action="{!addPresenceMapping}" status="customSettingStatus"
                                            reRender="addPresenceMappingBtn, presenceMappingsPanel, messages" disabled="{!IF(disableAddBtn,true,false)}"
                                            styleClass="slds-button slds-button--neutral" />
                                          <img width="15" height="15" src="{!URLFOR($Resource.AgentConsoleApp,'css/images/help.png')}" title="{! $Label.maxMappingsCanAdd}"
                                          />
                                        </span>
                                        <apex:outputPanel id="parentPresenceMappingsPanel">
                                          <apex:outputPanel id="presenceMappingsPanel" rendered="{!isOAuthInfoSet}">
                                            <div id="agentSyncStateUn" class="agentSyncStateUn" style="margin-top:6px;">
                                              <div style="padding-top: 2px;display:{!IF(AND(isMasterApp == 'Salesforce', sfPresenceMappings.size>0),'block','none')};"
                                                id="inContactState">
                                                <table class="entity-table presence-mapping-table">
                                                  <tr class="default-background">
                                                    <td style="padding-left:12px;"></td>
                                                    <td style="font-weight:bold;">{!$Label.sfOmniChannelStatus}</td>
                                                    <td></td>
                                                    <td style="font-weight:bold;">{!$Label.inContactStatus}</td>
                                                  </tr>
                                                  <apex:variable value="{!0}" var="index2" />
                                                  <apex:repeat value="{!sfPresenceMappings}" var="sfPresenceMapping">
                                                    <tr>
                                                      <td class="remove-mapping-btn">
                                                        <apex:commandButton value="{!$Label.icagentconsole__remove}" action="{!removePresenceMapping}" status="customSettingStatus" reRender="addPresenceMappingBtn, presenceMappingsPanel, messages"
                                                          styleClass="slds-button slds-button--neutral">
                                                          <apex:param name="sfPresenceMappingIndex" assignTo="{!removeSFPresenceMappingIndex}" value="{!index2}" />
                                                        </apex:commandButton>
                                                        <apex:variable value="{!index2 + 1}" var="index2" />
                                                      </td>
                                                      <td class="codesList">
                                                        <div class="slds-form-element__row">
                                                          <div class="slds-form-element">
                                                            <div class="slds-form-element__control">
                                                              <div class="slds-select_container">
                                                                <apex:selectList styleClass="slds-select selectCodes" multiselect="false" value="{!sfPresenceMapping.sfStatus}" size="1"
                                                                  disabled="{!IF(SFCodes.size==0,true,false)}">
                                                                  <apex:selectOption itemValue="--SELECT--"></apex:selectOption>
                                                                  <apex:selectOptions value="{!SFCodes}"></apex:selectOptions>
                                                                </apex:selectList>
                                                              </div>
                                                            </div>
                                                          </div>
                                                        </div>
                                                      </td>
                                                      <td class="mapsTo">=</td>
                                                      <td class="codesList">
                                                        <div class="slds-form-element__row">
                                                          <div class="slds-form-element">
                                                            <div class="slds-form-element__control">
                                                              <div class="slds-select_container">
                                                                <apex:selectList styleClass="slds-select selectCodes" multiselect="false" value="{!sfPresenceMapping.inContactStatus}" size="1"
                                                                  disabled="{!IF(InContactCodes.size==0,true,false)}">
                                                                  <apex:selectOption itemValue="--SELECT--"></apex:selectOption>
                                                                  <apex:selectOptions value="{!InContactCodes}"></apex:selectOptions>
                                                                </apex:selectList>
                                                              </div>
                                                            </div>
                                                          </div>
                                                        </div>
                                                      </td>
                                                    </tr>
                                                  </apex:repeat>
                                                </table>
                                              </div>

                                              <div style="display:{!IF(AND(isMasterApp == 'inContact', inContactPresenceMappings.size>0),'block','none')}">
                                                <table class="entity-table presence-mapping-table">
                                                  <tr class="default-background">
                                                    <td style="padding-left:12px;"></td>
                                                    <td style="font-weight:bold;">{!$Label.inContactStatus}</td>
                                                    <td></td>
                                                    <td style="font-weight:bold;">{!$Label.sfOmniChannelStatus}</td>
                                                  </tr>
                                                  <apex:variable value="{!0}" var="index1" />
                                                  <apex:repeat value="{!inContactPresenceMappings}" var="icPresenceMapping">
                                                    <tr>
                                                      <td class="remove-mapping-btn">
                                                        <apex:commandButton value="{!$Label.icagentconsole__remove}" action="{!removePresenceMapping}" status="customSettingStatus" reRender="addPresenceMappingBtn, presenceMappingsPanel, messages"
                                                          styleClass="slds-button slds-button--neutral">
                                                          <apex:param name="icPresenceMappingIndex" assignTo="{!removeICPresenceMappingIndex}" value="{!index1}" />
                                                        </apex:commandButton>
                                                        <apex:variable value="{!index1 + 1}" var="index1" />
                                                      </td>
                                                      <td class="codesList">
                                                        <div class="slds-form-element__row">
                                                          <div class="slds-form-element">
                                                            <div class="slds-form-element__control">
                                                              <div class="slds-select_container">
                                                                <apex:selectList styleClass="slds-select selectCodes" multiselect="false" value="{!icPresenceMapping.inContactStatus}" size="1"
                                                                  disabled="{!IF(InContactCodes.size==0,true,false)}">
                                                                  <apex:selectOption itemValue="--SELECT--"></apex:selectOption>
                                                                  <apex:selectOptions value="{!InContactCodes}"></apex:selectOptions>
                                                                </apex:selectList>
                                                              </div>
                                                            </div>
                                                          </div>
                                                        </div>
                                                      </td>
                                                      <td class="mapsTo">=</td>
                                                      <td class="codesList">
                                                        <div class="slds-form-element__row">
                                                          <div class="slds-form-element">
                                                            <div class="slds-form-element__control">
                                                              <div class="slds-select_container">
                                                                <apex:selectList styleClass="slds-select selectCodes" multiselect="false" value="{!icPresenceMapping.sfStatus}" size="1"
                                                                  disabled="{!IF(SFCodes.size==0,true,false)}">
                                                                  <apex:selectOption itemValue="--SELECT--"></apex:selectOption>
                                                                  <apex:selectOptions value="{!SFCodes}"></apex:selectOptions>
                                                                </apex:selectList>
                                                              </div>
                                                            </div>
                                                          </div>
                                                        </div>
                                                      </td>
                                                    </tr>
                                                  </apex:repeat>
                                                </table>
                                              </div>
                                            </div>
                                          </apex:outputPanel>
                                        </apex:outputPanel>
                                      </div>
                                    </div>
                                  </div>
                                </td>
                              </tr>

                              <!--ED-->

                              <tr>
                                <td>
                                  <apex:outputPanel style="padding-top: 15px;" id="taskCreationSection">
                                    <span style="font-weight: bold;">{!$Label.taskCreation}</span>
                                    <br/>
                                    <br/>
                                    <table id='taskCreationConfigTbl'>
                                      <tr>
                                        <th>{!$Label.channel}</th>
                                        <th>{!$Label.createTask}</th>
                                        <th>{!$Label.popTask}</th>
                                        <th>{!$Label.alwaysCreateTask}</th>
                                        <th>{!$Label.taskforRefused}</th>
                                        <th>{!$Label.elevationToPhone}</th>
                                        <th>{!$Label.elevationToEmail}</th>
                                        <th>{!$Label.elevationToSms}</th>
                                      </tr>
                                      <apex:repeat value="{!taskCreationSettings}" var="row">
                                        <tr>
                                          <td>{!row.localizedChannelName}</td>
                                          <td>
                                            <label class="slds-checkbox slds-checkbox_standalone">
                                              <apex:inputCheckbox value="{!row.createTask}" onchange="taskCreationSettingChange()" />
                                              <span class="slds-checkbox--faux"></span>
                                            </label>
                                          </td>
                                          <td>
                                            <label class="slds-checkbox slds-checkbox_standalone">
                                              <apex:inputCheckbox value="{!row.popTask}" disabled="{!IF(row.createTask == false, true, false)}" />
                                              <span class="slds-checkbox--faux"></span>
                                            </label>
                                          </td>
                                          <td>
                                            <label class="slds-checkbox slds-checkbox_standalone">
                                              <apex:inputCheckbox value="{!row.alwaysNewTask}" disabled="{!IF(row.createTask == false, true, false)}" />
                                              <span class="slds-checkbox--faux"></span>
                                            </label>
                                          </td>
                                          <td>
                                            <label class="slds-checkbox slds-checkbox_standalone">
                                              <apex:inputCheckbox value="{!row.taskforRefused}" disabled="{!IF(row.createTask == false, true, false)}" />
                                              <span class="slds-checkbox--faux"></span>
                                            </label>
                                          </td>
                                          <td>
                                            <label class="slds-checkbox slds-checkbox_standalone">
                                              <apex:inputCheckbox value="{!row.taskForElevatedPhone}" disabled="{!IF((row.channelName == 'Phone' || row.channelName == 'Digital' || row.createTask == false), true, false)}"
                                              />
                                              <span class="slds-checkbox--faux"></span>
                                            </label>
                                          </td>
                                          <td>
                                            <label class="slds-checkbox slds-checkbox_standalone">
                                              <apex:inputCheckbox value="{!row.taskForElevatedEmail}" disabled="{!IF((row.channelName == 'Email' || row.channelName == 'Digital' || row.createTask == false), true, false)}"
                                              />
                                              <span class="slds-checkbox--faux"></span>
                                            </label>
                                          </td>
                                          <td>
                                            <label class="slds-checkbox slds-checkbox_standalone">
                                              <apex:inputCheckbox value="{!row.taskForElevatedSms}" disabled="{!IF((row.channelName == 'SMS' || row.channelName == 'Digital' || row.createTask == false), true, false)}"
                                              />
                                              <span class="slds-checkbox--faux"></span>
                                            </label>
                                          </td>
                                        </tr>
                                      </apex:repeat>
                                    </table>
                                    <apex:commandButton title="{!$Label.icagentconsole__resettodefault}" value="{!$Label.icagentconsole__resettodefault}" action="{!resetTaskCreationSettings}"
                                      reRender="taskCreationSection" status="customSettingStatus" styleClass="slds-button slds-button--neutral btn"
                                    />
                                  </apex:outputPanel>
                                </td>
                              </tr>
                              <tr>
                                    <td style="width:50%;">
                                        <div style="padding-top: 6px;">
                                            <apex:outputPanel style="padding-top: 10px;" id="popBlock">
                                                <div>
                                                    <div style="padding-top: 10px; width:100%;">
                                                        <div class="slds-form-element__row">
                                                            <div class="slds-form-element">
                                                                <div class="slds-form-element__control">
                                                                    <label class="slds-checkbox" for="chkConfigureSearch">
                                                                        <input name="default" type="checkbox" id="chkConfigureSearch" onchange="onCheckboxChange('search', this.checked);" value="{!canConfigureSearch}"/>
                                                                        <span class="slds-checkbox--faux"></span>
                                                                        <span class="slds-form-element__label">{!$Label.configurableSearch}</span>
                                                                    </label>
                                                                </div>
                                                            </div>
                                                        </div>
                                                    </div>
                                                    <div style="padding-top: 10px; width:100%;">
                                                        <span style="font-weight: bold;">{!$Label.inContactDataStorage}</span>
                                                        <div style="padding-top: 10px; width:100%;">
                                                            <div class="slds-form-element__row">
                                                                <div class="slds-form-element">
                                                                    <div class="slds-form-element__control">
                                                                        <label class="slds-checkbox" for="chkStoreVar">
                                                                            <input name="default" type="checkbox" id="chkStoreVar" onchange="onCheckboxChange('data', this.checked);" value="{!canStoreVaribles}"/>
                                                                            <span class="slds-checkbox--faux"></span>
                                                                            <span class="slds-form-element__label">{!$Label.storeIncontactScriptVar}</span>
                                                                        </label>
                                                                    </div>
                                                                </div>
                                                            </div>
                                                        </div>
                                                    </div>
                                                </div>
                                            </apex:outputPanel>
                                        </div>
                                    </td>
                                </tr>
              <tr>
              <td style="width:100%;"  colspan="2">
                <div style="padding-top: 15px;">
                  <span style="font-weight: bold;">{!$Label.textToIconSetting}</span>
                  <div style="padding-top: 5px; width:100%;">
                    <div class="slds-form-element__row">
                      <div class="slds-form-element">
                        <div class="slds-form-element__control">
                          <label class="slds-radio" for="textToIconSettingsText">
                            <input name="textToIconSettings" type="radio" id="textToIconSettingsText" onchange="onCheckboxChange('isIcon', 0);"  style="transform: translateY(20%);"/>
                            <span class="slds-radio--faux"></span>
                            <span class="slds-form-element__label">{!$Label.textToIconSettingsText}</span>
                            <img width="15" height="15" src="{!URLFOR($Resource.AgentConsoleApp,'css/images/help.png')}" title="{! $Label.textToIconSettingsTextTooltip}"/>
                          </label>
                        </div>

                        <div class="slds-form-element__control">
                          <label class="slds-radio" for="textToIconSettingsIcon">
                            <input name="textToIconSettings" type="radio" id="textToIconSettingsIcon" onchange="onCheckboxChange('isIcon', 1);"  style="transform: translateY(20%);"/>
                            <span class="slds-radio--faux"></span>
                            <span class="slds-form-element__label">{!$Label.textToIconSettingsIcon}</span>
                            <img width="15" height="15" src="{!URLFOR($Resource.AgentConsoleApp,'css/images/help.png')}" title="{! $Label.textToIconSettingsIconTooltip}"/>
                          </label>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </td>
            </tr>
          
          <tr>
            <td style="width:100%;" colspan="2">
              <div style="padding-top: 15px;">
                <span style="font-weight: bold;">{!$Label.entityMappingType}</span>
                <div style="padding-top: 5px; width:100%;">
                  <div class="slds-form-element__row">
                    <div class="slds-form-element">
                      <div class="slds-form-element__control">
                        <label class="slds-radio" for="whowhatMappingsMulti">
                          <input name="whowhatMappings" type="radio" id="whowhatMappingsMulti" onchange="onCheckboxChange('whowhat', 0);" style="transform: translateY(20%);"
                          />
                          <span class="slds-radio--faux"></span>
                          <span class="slds-form-element__label">{!$Label.entityMappingTypeMulti}</span>
                          <img width="15" height="15" src="{!URLFOR($Resource.AgentConsoleApp,'css/images/help.png')}" title="{! $Label.entityMapMultiTooltip}"
                          />
                        </label>
                      </div>

                      <div class="slds-form-element__control">
                        <label class="slds-radio" for="whowhatMappingsSingle">
                          <input name="whowhatMappings" type="radio" id="whowhatMappingsSingle" onchange="onCheckboxChange('whowhat', 1);" style="transform: translateY(20%);"
                          />
                          <span class="slds-radio--faux"></span>
                          <span class="slds-form-element__label">{!$Label.entityMappingTypeSingle}</span>
                          <img width="15" height="15" src="{!URLFOR($Resource.AgentConsoleApp,'css/images/help.png')}" title="{! $Label.entityMapSingleTooltip}"
                          />
                        </label>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </td>
          </tr>
          <!--SF-6011-->
          <tr>
            <td>
             <div class="slds-form-element__control">
               <label class="slds-checkbox" for="fullIntegratedLightningExp">
                 <input name="whatMappings" type="checkbox" id="fullIntegratedLightningExp" value="{!isRelatesToConfigurable}" onchange="toggleDuelpicklist(this.checked)"  style="transform: translateY(20%);"/>
                 <span class="slds-checkbox--faux"></span>
                 <span class="slds-form-element__label limitContactMappingLabel">{!$Label.entityMappingTypeLightning}</span>
                 <img width="15" height="15" src="{!URLFOR($Resource.AgentConsoleApp,'css/images/help.png')}" title="{!$Label.entityMapFullyIntegratedTooltip}"/>
               </label>
             </div>
             </td>
           </tr>
           <tr id="duelpicklist" style="display:none;">
             <td style="padding-top: 15px;">
                <c:incMultiselectPicklist leftLabel="{!$Label.icagentconsole__available}"
                     leftOption="{!availableObjects}"
                     rightLabel="{!$Label.icagentconsole__selected}"
                     rightOption="{!selectedObjects}"
                     size="14"
                     width="150px"/>
             </td>
           </tr>
           <!--SF-6011-->
            <tr>
            <td colspan="2">
              <apex:actionStatus id="entityCollectionStatus">
                <apex:facet name="start">
                  <div class="slds-spinner_container">
                    <div class="slds-spinner--brand slds-spinner slds-spinner--large" role="alert">
                      <span class="slds-assistive-text"></span>
                      <div class="slds-spinner__dot-a"></div>
                      <div class="slds-spinner__dot-b"></div>
                    </div>
                  </div>
                  <div>
                    <div class="loading-layer" />
                    <div class="loading-panel">
                      <span class="loading-icn"></span>
                      <span class="loading-txt">{!$Label.processing}...</span>
                    </div>
                  </div>
                </apex:facet>
                <apex:facet name="stop">
                  <apex:outputPanel id="entityCollectionPanel">
                    <table width="100%;" style="margin-top: 14px;">
                      <tr>
                        <td>
                          <div class="slds-form-element__row" style="float: left;">
                            <div class="slds-form-element" style="width:200px">
                              <div class="slds-form-element__control">
                                <div class="slds-select_container">
                                  <select id="entityNames" multiselect="false" size="1" style="width: 180px; height: 23px;" styleClass="slds-select">
                                  </select>
                                </div>
                              </div>
                            </div>
                          </div>
                          <div style="float: left; padding-left: 10px;">
                            <apex:commandButton action="{!addEntity}" status="entityCollectionStatus" value="{!$Label.icagentconsole__addentity}" reRender="pnlFieldContainer, messages"
                              styleClass="slds-button slds-button--neutral" />
                          </div>
                        </td>
                        <td>
                        </td>
                      </tr>
                      <tr>
                        <td colspan="2">
                          <br/>
                        </td>
                      </tr>
                      <tr>
                        <td colspan="2">
                          <div class="message infoM4 info-panel">
                            <table class="messageTable" cellspacing="0" cellpadding="0" border="0" style="padding:0px;margin:0px;">
                              <tr>
                                <td>
                                  <!--<img class="msgIcon" title="info" src="{!URLFOR($Resource.icAgentConsole__AgentConsoleApp,'css/images/lightning/info.png')}" alt="info" />-->
                                  <apex:image styleClass="msgIcon" title="info" rendered="{!IsLightningMode}" url="{!URLFOR($Resource.icAgentConsole__AgentConsoleApp,'css/images/lightning/info.png')}"
                                  />
                                  <apex:image styleClass="msgIcon" title="info" rendered="{!IF($User.UITheme == 'Theme3' , true, false)}" url="{!URLFOR($Resource.icAgentConsole__AgentConsoleApp,'css/images/info.png')}"
                                  />
                                </td>
                                <td class="messageCell">
                                  <div class="messageText">
                                    {!$Label.configWarning}
                                  </div>
                                </td>
                              </tr>
                            </table>
                          </div>
                        </td>
                      </tr>
                      <tr>
                        <td colspan="2">
                          <br/>
                        </td>
                      </tr>
                      <tr>
                        <td colspan="2">
                          <apex:outputPanel id="pnlFieldContainer">
                            <apex:repeat value="{!entityDetails}" var="entity">
                              <div class="remove-border bPageBlock brandSecondaryBrd apexDefaultPageBlock secondaryPalette">
                                <apex:outputPanel id="pnlInfoItems">
                                  <apex:commandButton value="{!$Label.icagentconsole__remove}" action="{!removeEntity}" reRender="pnlFieldContainer, messages" styleClass="slds-button slds-button--neutral cus-field-entity">
                                    <apex:param name="EntityName" assignTo="{!entityName}" value="{!entity.entityName}" />
                                  </apex:commandButton>&nbsp;&nbsp;
                                  <span style="font-weight:bold;font-size:large;padding-left:20px;"> {!entity.entityLabel} &nbsp; &nbsp;&nbsp; &nbsp;</span>
                                  <apex:commandButton value="{!$Label.icagentconsole__addfield}" status="entityCollectionStatus" action="{!addEntityField}" reRender="pnlFieldContainer, messages"
                                    styleClass="slds-button slds-button--neutral">
                                    <apex:param name="EntityName" assignTo="{!entityName}" value="{!entity.entityName}" />
                                  </apex:commandButton>
                                  <apex:variable value="{!0}" var="index" />
                                  <table class="entity-table">
                                    <apex:repeat value="{!entity.configFields}" var="field">
                                      <tr>
                                        <td style="padding-left:10px; width:5%;">
                                          <apex:commandButton id="btnRemoveField" Value="{!$Label.icagentconsole__remove}" action="{!removeEntityField}" reRender="pnlFieldContainer, messages"
                                            styleClass="slds-button slds-button--neutral">
                                            <apex:param name="EntityName" assignTo="{!entityName}" value="{!entity.entityName}" />
                                            <apex:param name="FieldIndex" assignTo="{!removeIndex}" value="{!index}" />
                                          </apex:commandButton>
                                          <apex:variable value="{!index + 1}" var="index" />
                                        </td>
                                        <td style="padding-left:10px; width:30%">
                                          <div class="slds-form-element__row">
                                            <div class="slds-form-element" style="width:100%;">
                                              <div class="slds-form-element__control">
                                                <div class="slds-select_container">
                                                  <apex:selectList value="{!field.fieldName}" size="1" multiselect="false" styleClass="slds-select">
                                                    <apex:selectOptions value="{!entity.entityFields}"></apex:selectOptions>
                                                  </apex:selectList>
                                                </div>
                                              </div>
                                            </div>
                                          </div>
                                        </td>
                                        <td style="width:2%;padding-left: 10px;">
                                          =
                                        </td>
                                        <td style="padding-left:10px; width:26%;">
                                          <div class="slds-form-element__row">
                                            <div class="slds-form-element">
                                              <div class="slds-form-element__control">
                                                <div class="slds-select_container">
                                                  <apex:selectList value="{!field.valueObject}" id="ddlValObjects" size="1" style="width:100%;" onchange="return onValueObjectChange(this)"
                                                    multiselect="false" styleClass="slds-select">
                                                    <apex:selectOption itemValue="--SELECT--"></apex:selectOption>
                                                    <apex:selectOptions value="{!ValueObjects}"></apex:selectOptions>
                                                  </apex:selectList>
                                                </div>
                                              </div>
                                            </div>
                                          </div>
                                        </td>
                                        <td style="width:2%;padding-left: 10px;">
                                          -
                                        </td>
                                        <td style="padding-left:10px; padding-right:10px; width:35%;">
                                          <apex:outputPanel id="ddlCallInfoItems" style="{!IF(field.valueObject == 'CallInfo','display:block;width:100%;','display:none;width:100%;')}">
                                            <div class="slds-form-element__row">
                                              <div class="slds-form-element">
                                                <div class="slds-form-element__control">
                                                  <div class="slds-select_container">
                                                    <apex:selectList value="{!field.valueFieldCall}" size="1" multiselect="false" styleClass="slds-select" style="width:100%;">
                                                      <apex:selectOptions value="{!CallInfoItems}"></apex:selectOptions>
                                                    </apex:selectList>
                                                  </div>
                                                </div>
                                              </div>
                                            </div>
                                          </apex:outputPanel>
                                          <apex:outputPanel id="txtScriptVariable" style="{!IF(field.valueObject  == 'ScriptVariable','display:block;width:100%','display:none;width:100%;')}">
                                            <div class="slds-form-element__row">
                                              <div class="slds-form-element slds-size--1-of-1">
                                                <apex:inputtext value="{!field.valueFieldScript}" id="txtScriptVariableHidden" styleClass="slds-input" style="width:100%;"
                                                />
                                              </div>
                                            </div>
                                          </apex:outputPanel>
                                          <apex:outputPanel id="ddlSMSInfoItems" style="{!IF(field.valueObject  == 'SMSInfo','display:block;width:100%;','display:none;width:100%;')}">
                                            <div class="slds-form-element__row">
                                              <div class="slds-form-element">
                                                <div class="slds-form-element__control">
                                                  <div class="slds-select_container">
                                                    <apex:selectList value="{!field.valueFieldSMS}" size="1" multiselect="false" styleClass="slds-select" style="width:100%;">
                                                      <apex:selectOptions value="{!SMSInfoItems}"></apex:selectOptions>
                                                    </apex:selectList>
                                                  </div>
                                                </div>
                                              </div>
                                            </div>
                                          </apex:outputPanel>
                                          <apex:outputPanel id="ddlWorkItems" style="{!IF(field.valueObject  == 'WorkItemInfo','display:block;width:100%;','display:none;width:100%;')}">
                                            <div class="slds-form-element__row">
                                              <div class="slds-form-element">
                                                <div class="slds-form-element__control">
                                                  <div class="slds-select_container">
                                                    <apex:selectList value="{!field.valueFieldWorkItem}" size="1" multiselect="false" styleClass="slds-select" style="width:100%;">
                                                      <apex:selectOptions value="{!WorkItemInfoItems}"></apex:selectOptions>
                                                    </apex:selectList>
                                                  </div>
                                                </div>
                                              </div>
                                            </div>
                                          </apex:outputPanel>
                                          <apex:outputPanel id="ddlSocialContactInfoItems" style="{!IF(field.valueObject  == 'SocialContactInfo','display:block;width:100%;','display:none;width:100%;')}">
                                            <div class="slds-form-element__row">
                                              <div class="slds-form-element">
                                                <div class="slds-form-element__control">
                                                  <div class="slds-select_container">
                                                    <apex:selectList value="{!field.valueFieldSocialContact}" size="1" multiselect="false" styleClass="slds-select" style="width:100%;">
                                                      <apex:selectOptions value="{!SocialContactInfoItems}"></apex:selectOptions>
                                                    </apex:selectList>
                                                  </div>
                                                </div>
                                              </div>
                                            </div>
                                          </apex:outputPanel>
                                          <apex:outputPanel id="ddlChatInfoItems" style="{!IF(field.valueObject  == 'ChatInfo','display:block;width:100%;','display:none;width:100%;')}">
                                            <div class="slds-form-element__row">
                                              <div class="slds-form-element">
                                                <div class="slds-form-element__control">
                                                  <div class="slds-select_container">
                                                    <apex:selectList value="{!field.valueFieldChat}" size="1" multiselect="false" styleClass="slds-select" style="width:100%;">
                                                      <apex:selectOptions value="{!ChatInfoItems}"></apex:selectOptions>
                                                    </apex:selectList>
                                                  </div>
                                                </div>
                                              </div>
                                            </div>
                                          </apex:outputPanel>
                                          <apex:outputPanel id="ddlEmailInfoItems" style="{!IF(field.valueObject  == 'EmailInfo','display:block;width:100%;','display:none;width:100%;')}">
                                            <div class="slds-form-element__row">
                                              <div class="slds-form-element">
                                                <div class="slds-form-element__control">
                                                  <div class="slds-select_container">
                                                    <apex:selectList value="{!field.valueFieldEmail}" size="1" multiselect="false" styleClass="slds-select" style="width:100%;">
                                                      <apex:selectOptions value="{!EmailInfoItems}"></apex:selectOptions>
                                                    </apex:selectList>
                                                  </div>
                                                </div>
                                              </div>
                                            </div>
                                          </apex:outputPanel>
                                          <apex:outputPanel id="ddlVoicemailInfoItems" style="{!IF(field.valueObject  == 'VoicemailInfo','display:block;width:100%;','display:none;width:100%;')}">
                                            <div class="slds-form-element__row">
                                              <div class="slds-form-element">
                                                <div class="slds-form-element__control">
                                                  <div class="slds-select_container">
                                                    <apex:selectList value="{!field.valueFieldVoicemail}" size="1" multiselect="false" styleClass="slds-select" style="width:100%;">
                                                      <apex:selectOptions value="{!VoicemailInfoItems}"></apex:selectOptions>
                                                    </apex:selectList>
                                                  </div>
                                                </div>
                                              </div>
                                            </div>
                                          </apex:outputPanel>
                                        </td>
                                      </tr>
                                    </apex:repeat>
                                  </table>
                                </apex:outputPanel>
                              </div>
                            </apex:repeat>
                          </apex:outputPanel>
                        </td>
                      </tr>
                    </table>
                    <script>
                      var j1102 = jQuery.noConflict();
                      var entityCombo = j1102("#entityNames").off("change", onEntityChange).on("change", onEntityChange);

                      loadEntities();

                      function loadEntities() {
                        var entityKeyValues = '{!JSENCODE(EntityKeyValues)}';
                        var entities = IC_Common.parseJSON(entityKeyValues);
                        var entitiesLength = entities.length;
                        var entityData;
                        var entityItem;
                        for (var i = 0; i < entitiesLength; i++) {
                          entityData = entities[i];
                          entityItem = j1102("<option>").text(entityData.Key).attr("value", entityData.Value);
                          entityCombo.append(entityItem);
                        }
                      }

                      function onEntityChange() {
                        var selectedEntity = entityCombo.find('option:selected').val();
                        if (selectedEntity) {
                          onEntitySelectionChange(selectedEntity);
                        }
                      }
                    </script>
                  </apex:outputPanel>
                </apex:facet>
              </apex:actionStatus>
            </td>
          </tr>
          </table>
          </apex:outputpanel>
          </apex:outputPanel>
          </apex:facet>
          </apex:actionStatus>
        </apex:outputPanel>
        </td>
        </tr>
        </table>

        </div>

        <apex:actionfunction name="actionProfileSelected" action="{!onProfileSelected}" status="updateProfileStatus" rerender="parentOutputPanel, scriptPanel"
          onComplete="loadEntities()">
        </apex:actionfunction>
        <apex:actionfunction name="actionSettingSelectionChanged" action="{!actionSettingSelectionChanged}" status="customSettingStatus"
          rerender="cutomProfileOptionPanel, messages, scriptPanel">
          <apex:param name="selectedProfileOption" assignTo="{!selectedProfileOption}" value="" />
        </apex:actionfunction>
        <apex:actionfunction name="resetAgentSyncSelectionJS" action="{!ResetAgentSyncSelection}" rerender="cutomProfileOptionPanel, messages, scriptPanel"
          status="updateProfileStatus">
          <apex:param name="isOmnichannelPresenceSyncEnabled" assignTo="{!isOmnichannelPresenceSyncEnabled}" value="" />
        </apex:actionfunction>
        <apex:actionfunction name="taskCreationSettingChange" action="{!taskCreationSettingChange}" reRender="taskCreationSection"
          status="customSettingStatus">
        </apex:actionfunction>
        <apex:actionfunction name="changeMasterJS" action="{!changeMaster}" rerender="cutomProfileOptionPanel, scriptPanel, messages"
          status="updateProfileStatus">
          <apex:param name="isMasterApp" assignTo="{!isMasterApp}" value="" />
        </apex:actionfunction>
        <apex:actionFunction action="{!onEntitySelectionChange}" name="onEntitySelectionChange" rerender="" status="entityCollectionStatus">
          <apex:param name="param1" value="" assignTo="{!selectedEntityName}" />
        </apex:actionFunction>
        <apex:actionFunction action="{!clearPageMessage}" name="clearPageMessage" rerender="messages" status="entityCollectionStatus">
        </apex:actionFunction>
        <apex:actionfunction name="saveConfigurationJS" action="{!saveConfiguration}" status="updateProfileStatus" reRender="parentOutputPanel,messages, scriptPanel" oncomplete="scrollTopAction()">
            <apex:param name="canStoreVaribles" assignTo="{!canStoreVaribles}" value=""/>
            <apex:param name="canConfigureSearch" assignTo="{!canConfigureSearch}" value=""/>
            <apex:param name="entityMappingType" assignTo="{!entityMappingType}" value=""/>
            <apex:param name="selectedWhatObjects" assignTo="{!selectedWhatObjects}" value=""/>
            <apex:param name="isRelatesToConfigurable" assignTo="{!isRelatesToConfigurable}" value=""/>
            <apex:param name="textToIcon" assignTo="{!textToIcon}" value=""/>
          </apex:actionfunction>
        </apex:outputPanel>
      </apex:facet>
    </apex:actionStatus>
    <apex:actionfunction name="setModeJS" action="{!setMode}" rerender="masterPanel, messages, scriptPanel">
      <apex:param name="sfMode" assignTo="{!sfMode}" value="" />
    </apex:actionfunction>
    <apex:actionfunction name="setOAuthInfo" action="{!setOAuthInfo}" status="updateProfileStatus" rerender="parentPresenceMappingsPanel,messages"
      oncomplete="setOAuthInfoCompleted()">
      <apex:param name="oAuthInfo" assignTo="{!oAuthInfo}" value="" />
    </apex:actionfunction>
    <apex:actionfunction name="showExceptionMessage" action="{!showPageMessage}" status="updateProfileStatus" rerender="messages">
      <apex:param name="exceptionMessage" assignTo="{!exceptionMessage}" value="" />
    </apex:actionfunction>
    <apex:actionfunction name="initAuthenticateSFCredentials" action="{!initAuthenticateSFCredentials}" status="updateProfileStatus"
      oncomplete="authenticateSFCredentials()"></apex:actionfunction>
    <apex:actionfunction name="reRenderCxoneRoutingPanel" action="{!reRenderCxoneRoutingPanel}" status="updateProfileStatus"
      rerender="cxoneRoutingPanel" oncomplete="setCxOneRouting()">
      <apex:param name="isCxoneRoutingLoaded" assignTo="{!isCxoneRoutingLoaded}" value="" />
    </apex:actionfunction>
    <apex:actionfunction name="saveSFOmniRoutingSettings" action="{!saveSFOmniRoutingSettings}" status="updateProfileStatus"
      rerender="cxoneRoutingPanel,messages" oncomplete="canSaveConfiguration()">
      <apex:param name="sfUsername" assignTo="{!sfOmniRoutingSettings.salesforceUsername}" value="" />
      <apex:param name="sfPassword" assignTo="{!sfOmniRoutingSettings.salesforcePassword}" value="" />
      <apex:param name="pushTopicName" assignTo="{!sfOmniRoutingSettings.salesforcePushTopic}" value="" />
      <apex:param name="pocId" assignTo="{!sfOmniRoutingSettings.pointOfContactId}" value="" />
      <apex:param name="isEnabled" assignTo="{!sfOmniRoutingSettings.isEnabled}" value="" />
    </apex:actionfunction>
  </apex:form>


  <script type="text/javascript">
    var chkDataStore = false,
    chkConfigureSearch = false,
    entityMappingType=0,
    sfMode = '',
    isRelatesToConfigurable = false,
    textToIcon=0;
     
    
    function saveConfiguration () {
      // validate and save cxone routing settings only when omni-channel enabled for the org
      var isOmniChannelEnabled = {!isOmniChannelEnabled};
      if (!isOmniChannelEnabled) {
        triggerSaveConfiguration();
      } else {
        var sfUsername = j1102("#sfUsername").val(),
            sfPassword = j1102("#sfPassword").val(),
            pushTopicName = j1102("#pushTopicName").val(),
            pocId = j1102("#pocId").val();
            

      // validate cxone routing fields only when it is enabled
      if (j1102("#enableCxOneRouting")[0].checked) {
        var errorStr = validateCxoneRoutingFields(sfUsername, sfPassword, pushTopicName, pocId);

        if (IC_Validation.isNotNullOrEmpty(errorStr)) {
          showExceptionMessage(errorStr);
          return;
        }
        // just to show spinner while validating the salforce credentials
        initAuthenticateSFCredentials();
      } else {
        saveSFOmniRoutingSettings(sfUsername, sfPassword, pushTopicName, pocId, false);
      }
    }
    }

    function validateCxoneRoutingFields(sfUsername, sfPassword, pushTopicName, pocId) {
      if (IC_Validation.isNullOrEmpty(sfUsername)) {
        return "{!JSENCODE($Label.usernamerequired)}";
      } else if (IC_Validation.isNullOrEmptyOrEmptySpace(sfUsername)) {
        return "{!JSENCODE($Label.usernameisnotvalid)}";
      }
      if (IC_Validation.isNullOrEmpty(sfPassword)) {
        return "{!JSENCODE($Label.passwordrequired)}";
      }
      if (IC_Validation.isNullOrEmpty(pushTopicName)) {
        return "{!JSENCODE($Label.pushTopicNameRequired)}";
      }
      if (IC_Validation.isNullOrEmpty(pocId)) {
        return "{!JSENCODE($Label.pocIdRequired)}";
      }
    }
    
  function triggerSaveConfiguration() {
    var selectedList = {};
           
      j1102(".rightList option").each(function(){
            selectedList[j1102(this).val()] = j1102(this)[0].label;
      });
      saveConfigurationJS(chkDataStore, chkConfigureSearch, entityMappingType, JSON.stringify(selectedList), isRelatesToConfigurable,textToIcon);
    }

    function onCheckboxChange(element, val) {
      if (element === 'data') {
        chkDataStore = val;
      } else if (element === 'search') {
        chkConfigureSearch = val;
      }
      else if (element ==='whowhat') {
        entityMappingType=val;
       }else if (element === 'isIcon') {
        textToIcon = val;}
    }
    function toggleDuelpicklist (checked){
      if(checked){
        isRelatesToConfigurable = true;
        j1102("#duelpicklist").css('display','grid');
      }else{
         isRelatesToConfigurable = false;
         j1102("#duelpicklist").css('display','none');
      }
    }
    function onValueObjectChange(selectList) {
      var element = selectList;
      var infoType = selectList.options[selectList.selectedIndex].value;
      var ddlCallInfo = selectList.id.replace('ddlValObjects', 'ddlCallInfoItems');
      var ddlWorkItemInfo = selectList.id.replace('ddlValObjects', 'ddlWorkItems');
      var ddlChatInfo = selectList.id.replace('ddlValObjects', 'ddlChatInfoItems');
      var ddlEmailInfo = selectList.id.replace('ddlValObjects', 'ddlEmailInfoItems');
      var ddlVoicemailInfo = selectList.id.replace('ddlValObjects', 'ddlVoicemailInfoItems');
      var ddlSocialContactInfo = selectList.id.replace('ddlValObjects', 'ddlSocialContactInfoItems');
      var txtScriptvar = element.id.replace('ddlValObjects', 'txtScriptVariable');
      var ddlSMSInfo = selectList.id.replace('ddlValObjects', 'ddlSMSInfoItems');
      var pnlInfoItem = element.id.replace('ddlValObjects', 'pnlInfoItems');
      var targetDropdown = '';

      document.getElementById(ddlEmailInfo).style.display = 'none';
      document.getElementById(ddlCallInfo).style.display = 'none';
      document.getElementById(ddlWorkItemInfo).style.display = 'none';
      document.getElementById(ddlChatInfo).style.display = 'none';
      document.getElementById(ddlVoicemailInfo).style.display = 'none';
      document.getElementById(ddlSMSInfo).style.display = 'none';
      document.getElementById(ddlSocialContactInfo).style.display = 'none';
      document.getElementById(txtScriptvar).style.display = 'none';

      if (infoType === 'ScriptVariable') {
        targetDropdown = txtScriptvar;
      }
      else if (infoType === 'CallInfo') {
        targetDropdown = ddlCallInfo;
      }
      else if (infoType === 'WorkItemInfo') {
        targetDropdown = ddlWorkItemInfo;
      }
      else if (infoType === 'SocialContactInfo') {
        targetDropdown = ddlSocialContactInfo;
      }
      else if (infoType === 'ChatInfo') {
        targetDropdown = ddlChatInfo;
      }
      else if (infoType === 'EmailInfo') {
        targetDropdown = ddlEmailInfo;
      }
      else if (infoType === 'VoicemailInfo') {
        targetDropdown = ddlVoicemailInfo;
      }
      else if (infoType === 'SMSInfo') {
        targetDropdown = ddlSMSInfo;
      }

      if (targetDropdown) {
        document.getElementById(targetDropdown).style.display = 'block';
        document.getElementById(targetDropdown).style.width = '100%';
      }
      return false;
    }

    function closeAlert() {
      j1102(".message-panel").hide();
      clearPageMessage();
    }
  </script>
  <!--Classic Theme-->
  <apex:outputPanel rendered="{!$User.UITheme == 'Theme3'}">
    <script>
      sfMode = 'Classic';
      setModeJS(sfMode);
      <apex:stylesheet value="{!URLFOR($Resource.icAgentConsole__AgentConsoleApp,'css/inContactAgentConfig.css')}" />
    </script>
  </apex:outputPanel>

  <!--Lightning Theme-->
  <apex:outputPanel rendered="{!IsLightningMode}">
    <script>
      sfMode = 'Lightning';
      setModeJS(sfMode);
    </script>
    <apex:stylesheet value="{!URLFOR($Resource.icAgentConsole__AgentConsoleApp,'css/lightning/salesforce-lightning-design-system-vf.min.css')}"
    />
    <apex:stylesheet value="{!URLFOR($Resource.icAgentConsole__AgentConsoleApp,'css/lightning/inContactAgentLightning.css')}" />
    <apex:stylesheet value="{!URLFOR($Resource.icAgentConsole__AgentConsoleApp,'css/lightning/inContactAgentConfigLightning.css')}" />
  </apex:outputPanel>

  <apex:outputPanel id="scriptPanel">
    <script type="text/javascript">
      var j1102 = jQuery.noConflict();
      var queryParams = IC_Common.parseQueryString(window.location.search);
      var logger = IC_Common.getLogger('AgentConfigPage');
      j1102(document).ready(function ($) {
        if (AuthenticationDataStore.isTokenExpired() === true) {
          j1102('.masterPanelStyle').hide();
          AuthenticationDataStore.redirectToLoginPage(false);
        }

        var oAuthInfo = AuthenticationDataStore.getOAuthInfo();
        if (IC_Validation.isNotNullOrEmpty(oAuthInfo)) {
          var oAuth2Token = {
            "access_token": oAuthInfo.accessToken,
            "expires_in": oAuthInfo.expiresIn,
            "refresh_token": oAuthInfo.refreshToken,
            "refresh_token_server_uri": oAuthInfo.refreshTokenUri,
            "resource_server_base_uri": oAuthInfo.baseUri,
            "access_token_time": oAuthInfo.accessTokenTime
          }
          setOAuthInfo(JSON.stringify(oAuth2Token));
        }
      });

      function setOAuthInfoCompleted() {
        var isCxoneRoutingLoaded = {!isCxoneRoutingLoaded      
};

        // Set store-screenpop-variable and configurable-search-for-PC-contact settings
        j1102('input[id$=chkStoreVar]').prop( "checked", {!canStoreVaribles});
        j1102('input[id$=chkConfigureSearch]').prop( "checked", {!canConfigureSearch});
        j1102('input[id$=fullIntegratedLightningExp]').prop( "checked", {!isRelatesToConfigurable});
        
    
        chkDataStore = {!canStoreVaribles};
        chkConfigureSearch = {!canConfigureSearch};
        isRelatesToConfigurable = {!isRelatesToConfigurable};

        setPresentSync();
        setMasterApp();
        setProfileOptions();
        j1102("input[name='isMasterApp']").on("change", function(e){
        changeMasterJS(''+j1102(this).val());
        });
        j1102("input[name='profileOptions']").on("click", function(e){
        actionSettingSelectionChanged(''+j1102(this).val());
        });
        setFocusOnLoad();
        setWhoWhatMappings();
        setTextToIconSettings();
        // re-render cxoneRoutingPanel only for the first time of setOAuthInfo oncomplete
        if (!isCxoneRoutingLoaded) {
          reRenderCxoneRoutingPanel(true);
        }
        if(j1102('input[id$=fullIntegratedLightningExp]')[0].checked){
          j1102("#duelpicklist")[0].style.display='grid';
        }else{
          j1102("#duelpicklist")[0].style.display='none';
        }
      }

      // set Enable NICE inContact routing for Salesforce Omni-Channel checkbox when cxoneRoutingPanel re-rendered
      function setCxOneRouting() {
        var isOmniChannelEnabled = {!isOmniChannelEnabled      
};
      if (isOmniChannelEnabled) {
        var enableCxOneRoutingEl = j1102('#enableCxOneRouting'),
          isEnabled = enableCxOneRoutingEl.val();
        if (isEnabled === "true") {
          enableCxOneRoutingEl.attr("checked", "checked");
        } else {
          enableCxOneRoutingEl.removeAttr("checked");
        }
        enableCxOneRoutingEl.trigger("change");
      }
      }

      function setFocusOnLoad() {
      }

      function setWhoWhatMappings() {
        entityMappingType = {!entityMappingType};
        j1102('input[id$=whowhatMappingsMulti]').prop( "checked", entityMappingType == 0);
        j1102('input[id$=whowhatMappingsSingle]').prop( "checked", entityMappingType == 1);
       }

      function setTextToIconSettings() {
        textToIcon = {!textToIcon};
        j1102('input[id$=textToIconSettingsText]').prop( "checked", textToIcon == 0);
        j1102('input[id$=textToIconSettingsIcon]').prop( "checked", textToIcon == 1);
      }
     
      function setPresentSync() {
        j1102('input[id$=chkAgentAppSync]').prop("checked", {!isOmnichannelPresenceSyncEnabled});
      }

      function setMasterApp() {
        j1102("input[name=isMasterApp][value='{!JSENCODE(isMasterApp)}']").prop('checked', true);
      }

      function setProfileOptions() {
        j1102("input[name=profileOptions][value='{!JSENCODE(selectedProfileOption)}']").prop('checked', true);
      }

      function scrollTopAction() {
        j1102("html, body").animate({ scrollTop: 0 }, "slow");
      }

      function onToggleCxOneRouting(isEnabled) {
        var sfUsernameEl = j1102("#sfUsername"),
          sfPasswordEl = j1102("#sfPassword"),
          pushTopicNameEl = j1102("#pushTopicName"),
          pocIdEl = j1102("#pocId");
        if (isEnabled) {
          sfUsernameEl.prop('disabled', false);
          sfPasswordEl.prop('disabled', false);
          pushTopicNameEl.prop('disabled', false);
          pocIdEl.prop('disabled', false);
        } else {
          sfUsernameEl.attr("disabled", "disabled");
          sfPasswordEl.attr("disabled", "disabled");
          pushTopicNameEl.attr("disabled", "disabled");
          pocIdEl.attr("disabled", "disabled");
        }
      }

      function onTogglePwdIcon() {
        var sfPassword = j1102("#sfPassword"),
          togglePwdIcon = j1102("#togglePwdIcon");

        if (sfPassword.attr("type") === "password") {
          sfPassword.attr("type", "text");
          togglePwdIcon.addClass("show-pwd");
        } else {
          sfPassword.attr("type", "password");
          togglePwdIcon.removeClass("show-pwd");
        }
      }

      function authenticateSFCredentials() {
        logger.debug('authenticateSFCredentials fired');
        var sfUsername = j1102("#sfUsername").val(),
          sfPassword = j1102("#sfPassword").val(),
          pushTopicName = j1102("#pushTopicName").val(),
          pocId = j1102("#pocId").val(),
          loginResult,
          pushTopicPermissionErrorMsg = "{!JSENCODE($Label.pushTopicPermissionError)}";

        try {
          loginResult = sforce.connection.login(sfUsername, sfPassword);
          logger.debug('authenticateSFCredentials loginResult : ' + loginResult);
          var query = "SELECT IsQueryable,IsRetrieveable FROM EntityDefinition WHERE DeveloperName = 'PushTopic'";
          var queryResult = sforce.connection.query(query);
          var records = queryResult.getArray('records');
          logger.debug('Push Topic access query result records count' + records.length);
          if (IC_Validation.isNotNullOrEmptyArray(records) && IC_Common.toBoolean(records[0].IsQueryable) && IC_Common.toBoolean(records[0].IsRetrieveable)) {
            saveSFOmniRoutingSettings(sfUsername, sfPassword, pushTopicName, pocId, true);
          } else {
            logger.debug('Entered Salesforce API User for NICE incontact routing does not have read permission to access the push topic');
            showExceptionMessage(pushTopicPermissionErrorMsg);
          }

        } catch (ex) {
          logger.debug('authenticateSFCredentials exception : ' + ex);
          if (ex.faultstring) {
            showExceptionMessage(ex.faultstring);
          }
        }
      }
    </script>
  </apex:outputPanel>
</apex:page>
