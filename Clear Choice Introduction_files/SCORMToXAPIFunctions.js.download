/*******************************************************************************
 **
 ** xapi object to be used in SCORM wrapper
 ** 
 ** Version 1.1
 **
 ** Converts many SCORM 2004 data model elements to associated xAPI data
 **
 *******************************************************************************/
xapi = function () {

    var _reservedQSParams = {
        //
        // these are TC spec reserved words that may end up in queries to the endpoint
        //
        statementId: true,
        voidedStatementId: true,
        verb: true,
        object: true,
        registration: true,
        context: true,
        actor: true,
        since: true,
        until: true,
        limit: true,
        authoritative: true,
        sparse: true,
        instructor: true,
        ascending: true,
        continueToken: true,
        agent: true,
        activityId: true,
        stateId: true,
        profileId: true,
        endpoint: true,
        auth: true,
        username: true,
        password: true,

        //
        // these are suggested by the LMS launch spec addition that TinCanJS consumes
        //
        activity_platform: true,
        grouping: true,
        "Accept-Language": true
    };

    var lrsProps = ["endpoint", "auth"];

    function convertToIso(secondsVar) {
        let seconds = secondsVar;
        if (seconds > 60) {
            if (seconds > 3600) {
                const hours = Math.floor(seconds / 3600);
                const minutes = Math.floor((seconds % 3600) / 60);
                seconds = (seconds % 3600) % 60;
                return 'PT' + hours + 'H' + minutes + 'M' + seconds + 'S';
            } else {
                const minutes = Math.floor(seconds / 60);
                seconds %= 60;
                return 'PT' + minutes + 'M' + seconds + 'S';
            }
        } else {
            return 'PT' + seconds + 'S';
        }
    }

    function parseURL(e) {
        var t, i, n, r, s = /\+/g, o = /([^&=]+)=?([^&]*)/g, a = function (e) {
            return decodeURIComponent(e.replace(s, " "))
        };
        if (t = new RegExp(["^(https?:)//", "(([^:/?#]*)(?::([0-9]+))?)", "(/[^?#]*)", "(\\?[^#]*|)", "(#.*|)$"].join("")),
            (n = {
                protocol: (i = e.match(t))[1],
                host: i[2],
                hostname: i[3],
                port: i[4],
                pathname: i[5],
                search: i[6],
                hash: i[7],
                params: {}
            }).path = n.protocol + "//" + n.host + n.pathname,
            "" !== n.search)
            for (; r = o.exec(n.search.substring(1));)
                n.params[a(r[1])] = a(r[2]);
        return n
    }

    var _debug = true;

    function closeMessage(e) {
        e = e || window.event;

        // For IE and Firefox prior to version 4
        if (e) {
            e.returnValue = 'Please click exit button in the menu before closing the window';
        }

        // For Safari
        return 'Please click exit button in the menu before closing the window';
    };


    /*******************************************************************************
     **
     ** Configuration object for a specific instance of the wrapper
     **
     ** The following configuration values must be set in order for this 
     ** wrapper to function correctly:
     **
     ** LRS Data
     ** -----------
     ** endpoint - Points at the LRS endpoint
     ** user -  Username for the LRS
     ** password - Password for the LRS
     **
     ** Other Configuration Values
     ** ----------------------------
     ** courseId - IRI for the course this wrapper is used in
     ** lmsHomePage - LMS home page where the course is/will be imported
     ** isSCORM2004 - Whether the original course is SCORM 2004 (or SCORM Version 1.2)
     ** activityId - The ID that will identify the SCO/Activity in the LRS
     ** groupingContextActivity - Context activity for a synchronous workshop (if applicable)
     **
     ** Note: DO NOT UPDATE THE "constants" below.  These are used to indentify
     **       SCORM profile information and should not be changed
     **
     *******************************************************************************/

    var envConfig = parseURL(window.location.href).params;
    var lrs = {};
    var i, extended = null;

    if (envConfig.hasOwnProperty('endpoint')) {
        lrs.endpoint = envConfig.endpoint;

        for (i in envConfig) {
            if (!_reservedQSParams.hasOwnProperty(i)) {
                extended = extended || {};
                extended[i] = envConfig[i];
            }
        }

    }

    if (envConfig.hasOwnProperty('username')) {
        lrs.username = envConfig.username;
    }

    if (envConfig.hasOwnProperty('password')) {
        lrs.password = envConfig.password;
    }

    if (envConfig.hasOwnProperty('auth')) {
        delete lrs.username;
        delete lrs.password;

        lrs.auth = envConfig.auth;
    }

    if (lrs.endpoint) {
        window.lrsConfig.lrs = lrs;
    }

    var grouping = envConfig.hasOwnProperty('grouping') ? envConfig.grouping : null;
    var registration = envConfig.registration;
    var actor = !envConfig.actor ? null : JSON.parse(envConfig.actor);

    if (!!actor && typeof actor.account === "object" && actor.account !== null && typeof actor.account.homePage === "undefined" && typeof actor.account.name === "undefined") {
        if (actor.account.length === 0) {
            delete actor.account;
        }
        else {
            actor.account = actor.account[0];
        }
    }

    if (!!actor && typeof actor.account === "object" && actor.account !== null && typeof actor.account.accountServiceHomePage !== "undefined") {
        actor.account.homePage = actor.account.accountServiceHomePage;
        delete actor.account.accountServiceHomePage;
    }

    if (!!actor && typeof actor.account === "object" && actor.account !== null && typeof actor.account.accountName !== "undefined") {
        actor.account.name = actor.account.accountName;
        delete actor.account.accountName;
    }

    if (!!actor && typeof actor.name === "object") {
        actor.name = actor.name[0];
    }

    if (!!actor && typeof actor.mbox === "object") {
        actor.mbox = actor.mbox[0];
    }

    var config = window.lrsConfig;

    /*******************************************************************************
     **
     ** This function is used to configure LRS endpoint and basic auth values
     **
     *******************************************************************************/
    var configureLRS = function () {
        var conf = !config.lrs.auth ? {
            endpoint: config.lrs.endpoint,
            user: config.lrs.user,
            password: config.lrs.password,
            registration: registration
        } : {
                endpoint: config.lrs.endpoint,
                auth: config.lrs.auth,
                registration: registration
            };

        if (extended !== null) {
            conf.extended = extended;
        }

        ADL.XAPIWrapper.changeConfig(conf);
    }

    /*******************************************************************************
     **
     ** Gets agent - account corresponding to LMS user registration
     ** 
     ** Used when accessing state objects
     **
     *******************************************************************************/
    var getAgent = function () {
        if (!!actor) {

            if (!actor.account && !actor.mbox) {
                actor.account = {
                    homePage: config.lmsHomePage,
                    name: actor.name
                }
            };

            return actor;
        }

        if (!window.localStorage.learnerId) {
            window.localStorage.learnerId = retrieveDataValue(scormVersionConfig.learnerIdElement);
        }

        var agent = {
            account: {
                homePage: config.lmsHomePage,
                name: window.localStorage.learnerId
            }
        };

        return agent;
    }

    // configure lrs
    configureLRS();
    var agent = getAgent();

    window.localStorage.learnerId = agent.account ? agent.account.name : (agent.mbox ? agent.mbox.replace('mailto:', '') : '');

    var bookmark = registration ? ADL.XAPIWrapper.getState(config.activityId, getAgent(), 'bookmark', registration) : null;
    var suspend_data = registration ? ADL.XAPIWrapper.getState(config.activityId, getAgent(), 'suspend_data', registration) : null;
    var cumulative_time = registration ? ADL.XAPIWrapper.getState(config.activityId, getAgent(), 'cumulative_time', registration) : 0;
    var startAtInSeconds = Math.floor(Date.now() / 1000);

    if (!cumulative_time) {
        cumulative_time = 0;
    } else {
        cumulative_time = parseInt(cumulative_time);
    }

    var data = {
        'cmi.core.student_id': name,
        'cmi.core.student_name': name,
        'cmi.core.lesson_location': '',
        'cmi.core.lesson_status': 'not attempted'
    };


    if (suspend_data) {
        data['cmi.suspend_data'] = suspend_data;
    }

    if (bookmark) {
        data['cmi.core.lesson_location'] = bookmark;
    }

    window.localStorage.setItem('data', JSON.stringify(data));

    // xAPI SCORM Profile IRI contstants
    // https://github.com/adlnet/xAPI-SCORM-Profile/blob/master/xapi-scorm-profile.md
    var constants = {
        activityProfileIri: "https://w3id.org/xapi/scorm/activity-profile",
        activityStateIri: "https://w3id.org/xapi/scorm/activity-state",
        actorProfileIri: "https://w3id.org/xapi/scorm/agent-profile",
        attemptStateIri: "https://w3id.org/xapi/scorm/attempt-state"
    };

    // used to hold the data model elements to be used based on SCORM Version
    var scormVersionConfig = {};

    // used to identify if a suspend occurred
    var exitSetToSuspend = false;

    /*******************************************************************************
     **
     ** Base statement
     ** 
     ** Must update verb, attempt and result (if applicable) to execute
     **
     *******************************************************************************/
    var getBaseStatement = function () {
        if (!window.localStorage.learnerId) {
            window.localStorage.learnerId = retrieveDataValue(scormVersionConfig.learnerIdElement);
        }

        var baseStatement = {
            actor: getAgent(),
            verb: {},
            object: {
                id: config.activityId,
                objectType: "Activity",
                definition: {
                    type: "http://adlnet.gov/expapi/activities/lesson"
                }
            },
            context: {
                contextActivities: {
                    grouping: [
                        {
                            id: "",
                            objectType: "Activity",
                            definition: {
                                type: "http://adlnet.gov/expapi/activities/attempt"
                            }
                        },
                        {
                            id: config.courseId,
                            objectType: "Activity",
                            definition: {
                                type: "http://adlnet.gov/expapi/activities/course"
                            }
                        }
                    ],
                    category: [
                        {
                            id: "https://w3id.org/xapi/scorm"
                        }
                    ]
                }
            }
        };
        if (grouping) {
            baseStatement.context.contextActivities.grouping[2] = {
                id: grouping,
                objectType: "Activity",
                definition: {
                    type: "http://adlnet.gov/expapi/activities/lms"
                }
            };
        }
        return baseStatement;
    }

    /*******************************************************************************
     **
     ** Interactions base statement
     ** 
     ** Must update object iri, attempt, result and interaction 
     ** type/description to execute
     **
     *******************************************************************************/
    var getInteractionsBaseStatement = function () {
        if (!window.localStorage.learnerId) {
            window.localStorage.learnerId = retrieveDataValue(scormVersionConfig.learnerIdElement);
        }

        var baseStatement = {
            actor: getAgent(),
            verb: ADL.verbs.responded,
            object: {
                objectType: "Activity",
                id: "",
                definition: {
                    type: "http://adlnet.gov/expapi/activities/cmi.interaction",
                    interactionType: "",
                    correctResponsesPattern: []
                }
            },
            context: {
                contextActivities: {
                    parent: [
                        {
                            id: config.activityId,
                            objectType: "Activity",
                            definition: {
                                type: "http://adlnet.gov/expapi/activities/lesson"
                            }
                        }
                    ],
                    grouping: [
                        {
                            id: "",
                            objectType: "Activity",
                            definition: {
                                type: "http://adlnet.gov/expapi/activities/attempt"
                            }
                        },
                        {
                            id: config.courseId,
                            objectType: "Activity",
                            definition: {
                                type: "http://adlnet.gov/expapi/activities/course"
                            }
                        }
                    ],
                    category: [
                        {
                            id: "https://w3id.org/xapi/scorm"
                        }
                    ]
                }
            },
            result: {
                response: ""
            }
        };

        if (grouping) {
            baseStatement.context.contextActivities.grouping[2] = {
                id: grouping,
                objectType: "Activity",
                definition: {
                    type: "http://adlnet.gov/expapi/activities/lms"
                }
            };
        }
        return baseStatement;
    }

    /*******************************************************************************
     **
     ** Voided base statement
     ** 
     ** Must set verb and object to execute
     **
     *******************************************************************************/
    var getVoidedBaseStatement = function () {
        if (!window.localStorage.learnerId) {
            window.localStorage.learnerId = retrieveDataValue(scormVersionConfig.learnerIdElement);
        }

        return {
            actor: getAgent(),
            verb: {},
            object: {
                objectType: "StatementRef",
                id: ""
            }
        };
    }

    /*******************************************************************************
     **
     ** This function is used to initiate an xAPI attempt
     **
     *******************************************************************************/
    var initializeAttempt = function () {
        // configure SCORM version and data elements, get launch data from lms, etc
        configureXAPIData();

        window.addEventListener('beforeunload', closeMessage);

        // deprecated - set the agent profile information based on LMS learner_prefernces
        //setAgentProfile();

        // todo: add error handling to SCORM call
        // Determine whether this is a new or resumed attempt (based on cmi.entry)
        var entry = retrieveDataValue(scormVersionConfig.entryElement);


        var isResumed = (entry == "resume");

        // if "resume", determine if the user issued a suspend sequencing nav 
        // request and a terminate was called instead of a suspend and if so, fix
        if (suspend_data) {
            adjustFinishStatementForResume();
        }

        // set the attempt context activity based on the SCOs state
        configureAttemptContextActivityID(suspend_data ? "resume" : "");

        // Set activity profile info and attempt state every initialize
        // todo: these cause acceptable errors.  ensure they are not written to console
        setActivityProfile();
        setAttemptState();

        // Set the appropriate verb based on resumed or new attempt
        var startVerb = suspend_data ? ADL.verbs.resumed : ADL.verbs.initialized;

        // Execute the statement
        sendSimpleStatement(startVerb);
    }

    /*******************************************************************************
     **
     ** This function looks at the last terminate or statement for a given attempt.
     ** If "terminated", the terminated stmt is voided and a suspend is issued
     **
     *******************************************************************************/
    var adjustFinishStatementForResume = function () {
        var search = ADL.XAPIWrapper.searchParams();
        search['verb'] = ADL.verbs.terminated.id;
        search['activity'] = config.activityId;
        search['related_activities'] = true;

        var res = ADL.XAPIWrapper.getStatements(search);

        if (res.statements.length == 1) {
            // there is a terminate verb, so must void it and replace with suspended
            // Note: if there is length == 0, no issue.  
            //       if length > 1, things are very messed up. Do nothing.

            var terminateStmt = res.statements[0];

            // send the voided statement
            var voidedStmt = getVoidedBaseStatement();
            voidedStmt.verb = ADL.verbs.voided;
            voidedStmt.object.id = terminateStmt.id;

            setTimeout(() => {
                var response = ADL.XAPIWrapper.sendStatement(voidedStmt);
            }, 0);

            // send a suspended statement to replace the (voided) terminated statement
            suspendAttempt(terminateStmt.timestamp);
        }

    }


    /*******************************************************************************
     **
     ** This function is used to resume an attempt
     **
     *******************************************************************************/
    var resumeAttempt = function () {
        sendSimpleStatement(ADL.verbs.resumed);
    }

    /*******************************************************************************
     **
     ** This function is used to suspent an attempt
     **
     *******************************************************************************/
    var suspendAttempt = function (timestamp) {
        //sendSimpleStatement(ADL.verbs.suspended);
        var stmt = getBaseStatement();
        var nowInSeconds = Math.floor(Date.now() / 1000);
        cumulative_time += nowInSeconds - startAtInSeconds;

        if (registration) {
            setTimeout(() => {
                ADL.XAPIWrapper.sendState(config.activityId, getAgent(), 'cumulative_time', registration, cumulative_time);
            }, 0);
        }

        stmt.verb = ADL.verbs.suspended;

        if (timestamp != undefined && timestamp != null) {
            stmt.timestamp = timestamp;
        }

        // window.localStorage[activity] uses activity id to return the most recent
        // attempt
        stmt.context.contextActivities.grouping[0].id = window.localStorage[config.activityId];

        // set the context activity from the manifest/launch_data to group together
        // for an event
        //stmt.context.contextActivities.grouping.push(config.groupingContextActivity);

        var stmtWithResult = getStmtWithResult(stmt);
        stmtWithResult.result.duration = convertToIso(cumulative_time);
        setTimeout(() => {
            var response = ADL.XAPIWrapper.sendStatement(stmtWithResult);
        }, 0);
        if (!timestamp) {
            window.removeEventListener('beforeunload', closeMessage);
            document.getElementsByTagName('body')[0].innerHTML = '<p style="text-align:center;">You can close window now<p>';
        }
    }

    /*******************************************************************************
     **
     ** This function is used to terminate an xAPI attempt
     **
     *******************************************************************************/
    var terminateAttempt = function () {
        //sendSimpleStatement(ADL.verbs.terminated);
        var stmt = getBaseStatement();
        var nowInSeconds = Math.floor(Date.now() / 1000);
        cumulative_time += nowInSeconds - startAtInSeconds;

        if (registration) {
            setTimeout(() => {
                ADL.XAPIWrapper.sendState(config.activityId, getAgent(), 'cumulative_time', registration, cumulative_time);
            }, 0);
        }

        // get the exit and use appropriate verb
        var stopVerb = (exitSetToSuspend) ? ADL.verbs.suspended : ADL.verbs.terminated;

        stmt.verb = stopVerb;

        // window.localStorage[activity] uses activity id to return the most recent
        // attempt
        stmt.context.contextActivities.grouping[0].id = window.localStorage[config.activityId];

        // set the context activity from the manifest/launch_data to group together
        // for an event
        //stmt.context.contextActivities.grouping.push(config.groupingContextActivity);

        var stmtWithResult = getStmtWithResult(stmt);
        stmtWithResult.result.duration = convertToIso(cumulative_time);
        setTimeout(() => {
            var response = ADL.XAPIWrapper.sendStatement(stmtWithResult);
        }, 0);

        window.localStorage.removeItem("learnerId");
        window.removeEventListener('beforeunload', closeMessage);
        document.getElementsByTagName('body')[0].innerHTML = '<p style="text-align:center;">You can close window now<p>';
    }

    /*******************************************************************************
     **
     ** This function is used to complete the stmt result for terminate and suspend
     **
     *******************************************************************************/
    var getStmtWithResult = function (baseStatement) {
        var success = retrieveDataValue(scormVersionConfig.successElement);
        var completion = retrieveDataValue(scormVersionConfig.completionElement);
        var scoreScaled = retrieveDataValue(scormVersionConfig.scoreScaledElement);
        var scoreRaw = retrieveDataValue(scormVersionConfig.scoreRawElement);
        var scoreMin = 0;
        var scoreMax = 100;

        if (completion === 'failed' && scoreRaw === '') {
            scoreRaw = "0";
        }

        var resultSet = false;
        var resultJson = {};
        var scoreSet = false;
        var scoreJson = {};

        // create all of the statement json 

        // set success if known
        if (success == "passed") {
            resultSet = true;
            resultJson.success = true;
        } else if (success == "failed") {
            resultSet = true;
            resultJson.success = false;
        }

        // set completion if known
        if (completion == "completed" || completion == "passed") {
            resultSet = true;
            resultJson.completion = true;
        } else {
            resultSet = true;
            resultJson.completion = false;
        }

        // set scaled score if set by sco
        if (scoreScaled != undefined && scoreScaled != "") {
            scoreSet = true;
            resultSet = true;
            scoreJson.scaled = parseFloat(scoreScaled);
        }

        // set raw score if set by sco
        if (scoreRaw != undefined && scoreRaw != "") {
            scoreSet = true;
            resultSet = true;
            scoreJson.raw = parseFloat(scoreRaw);

            // if SCORM 1.2, use raw score / 100 for scaled score
            if (!config.isScorm2004) {
                scoreJson.scaled = parseFloat(scoreRaw) / 100;
            }
        }

        // set min score if set by sco
        if (scoreMin != undefined && scoreMin != "") {
            scoreSet = true;
            resultSet = true;
            scoreJson.min = parseFloat(scoreMin);
        }

        // set max score if set by sco
        if (scoreMax != undefined && scoreMax != "") {
            scoreSet = true;
            resultSet = true;
            scoreJson.max = parseFloat(scoreMax);
        }

        // set the score object in with the rest of the result object
        if (scoreSet) {
            resultJson.score = scoreJson;
        }

        // add result to the base statement
        if (resultSet) {
            baseStatement.result = resultJson;
        }

        return baseStatement;
    }

    /*******************************************************************************
     **
     ** This function is used to set agent data based on SCORM learner prefs
     **
     ** Deprecated
     **
     *******************************************************************************/
    var setAgentProfile = function () {

        if (!window.localStorage.learnerId) {
            window.localStorage.learnerId = retrieveDataValue(scormVersionConfig.learnerIdElement);
        }

        var lang = retrieveDataValue(scormVersionConfig.languageElement);
        var audioLevel = retrieveDataValue(scormVersionConfig.audioLevelElement);
        var deliverySpeed = retrieveDataValue(scormVersionConfig.deliverySpeedElement);
        var audioCaptioning = retrieveDataValue(scormVersionConfig.audioCaptioningElement);

        var profile = {
            language: lang,
            audio_level: audioLevel,
            delivery_speed: deliverySpeed,
            audio_captioning: audioCaptioning
        };

        ADL.XAPIWrapper.sendAgentProfile({
            account: getAgent().account
        },
            config.activityId,
            profile,
            null,
            "*"
        );
    }

    /*******************************************************************************
     **
     ** This function is used to set activity profile information 
     **
     ** Note: this data is scoped to an activity and does not (normally) change
     **
     *******************************************************************************/
    var setActivityProfile = function () {
        // see if the profile is already set
        try {
            var ap = ADL.XAPIWrapper.getActivityProfile(config.activityId, constants.activityProfileIri);

            if (!ap) {
                // get comments from lms (if any)
                //var cmi_num_comments_from_lms_count = retrieveDataValue("cmi.comments_from_lms._count");
                // todo: get the comments, if any and add to array

                // get completion threshold (if supplied in manifest)
                var cmi_completion_threshold = retrieveDataValue(scormVersionConfig.completionThresholdElement);
                var cmi_launch_data = retrieveDataValue(scormVersionConfig.launchDataElement);
                var cmi_max_time_allowed = retrieveDataValue(scormVersionConfig.maxTimeAllowedElement);
                var cmi_scaled_passing_score = retrieveDataValue(scormVersionConfig.scaledPassingScoreElement);
                var cmi_time_limit_action = retrieveDataValue(scormVersionConfig.timeLimitActionElement);

                var activityProfile = {};

                if (config.isScorm2004 && cmi_completion_threshold != "")
                    activityProfile.completion_threshold = cmi_completion_threshold;

                if (cmi_launch_data != "")
                    activityProfile.launch_data = cmi_launch_data;

                if (cmi_max_time_allowed != "")
                    activityProfile.max_time_allowed = cmi_max_time_allowed;

                if (cmi_scaled_passing_score != "")
                    activityProfile.scaled_passing_score = cmi_scaled_passing_score;

                if (cmi_time_limit_action != "")
                    activityProfile.time_limit_action = cmi_time_limit_action;


                ADL.XAPIWrapper.sendActivityProfile(config.activityId, constants.activityProfileIri, activityProfile, null, "*");
            }
        } catch (e) {
            console.warn('cannot get profile')
        }

    }

    /*******************************************************************************
     **
     ** This function is used to set activity state
     **
     ** Note: State data about an activity that is different for each user
     **
     **       This is used to also update attempt iri array associated with 
     **       the user and activity
     **
     *******************************************************************************/
    var setActivityState = function () {
        // window.localStorage[activity] uses activity id to return the most recent
        // attempt
        var attemptIri = window.localStorage[config.activityId];

        var agent = getAgent();

        // see if the profile is already set
        var as = ADL.XAPIWrapper.getState(config.activityId, agent, constants.activityStateIri);

        // First time, create a new one
        if (!as) {
            setTimeout(() => {
                ADL.XAPIWrapper.sendState(config.activityId, agent, constants.activityStateIri, null, {
                    attempts: [attemptIri]
                });
            }, 0);
        } else {
            // update state
            var asStr = JSON.stringify(as)
            var newAs = JSON.parse(asStr);

            newAs.attempts.push(attemptIri);

            setTimeout(() => {
                ADL.XAPIWrapper.sendState(config.activityId, agent, constants.activityStateIri, null, newAs, ADL.XAPIWrapper.hash(asStr));
            }, 0);
        }
    }

    /*******************************************************************************
     **
     ** This function is used to set attempt (activity) state
     **
     ** Note: State data about an activity that is different for each user, for each
     **       attempt.
     **
     *******************************************************************************/
    var setAttemptState = function () {
        // window.localStorage[activity] uses activity id to return the most recent
        // attempt
        var attemptIri = window.localStorage[config.activityId];
        var agent = getAgent();

        // location, preferences object, credit, lesson_mode, suspend_data, 
        // total_time, adl_data
        var cmi_location = retrieveDataValue(scormVersionConfig.locationElement);

        var cmi_language = retrieveDataValue(scormVersionConfig.languageElement);
        var cmi_audio_level = retrieveDataValue(scormVersionConfig.audioLevelElement);
        var cmi_delivery_speed = retrieveDataValue(scormVersionConfig.deliverySpeedElement);
        var cmi_audio_captioning = retrieveDataValue(scormVersionConfig.audioCaptioningElement);

        var preferences = {
            language: cmi_language,
            audio_level: cmi_audio_level,
            delivery_speed: cmi_delivery_speed,
            audio_captioning: cmi_audio_captioning
        };

        var cmi_credit = retrieveDataValue(scormVersionConfig.creditElement);
        var cmi_mode = retrieveDataValue(scormVersionConfig.modeElement);
        var cmi_suspend_data = retrieveDataValue(scormVersionConfig.suspendDataElement);
        var cmi_total_time = retrieveDataValue(scormVersionConfig.totalTimeElement);

        // todo: implement adl.data buckets and store in attempt state

        // create the state object
        var state = {};

        if (cmi_location != "")
            state.location = cmi_location;

        state.preferences = preferences;

        if (cmi_credit != "")
            state.credit = cmi_credit;

        if (cmi_mode != "")
            state.mode = cmi_mode;

        if (cmi_suspend_data != "")
            state.suspend_data = cmi_suspend_data;

        if (cmi_total_time != "")
            state.total_time = cmi_total_time;



        // see if the profile is already set
        var as = ADL.XAPIWrapper.getState(attemptIri, agent, constants.attemptStateIri);

        if (!as) {
            // first set on this attempt
            setTimeout(() => {
                ADL.XAPIWrapper.sendState(attemptIri, agent, constants.attemptStateIri, null, state);
            }, 0);
        } else {
            var asStr = JSON.stringify(as);

            // updating existing attempt
            setTimeout(() => {
                ADL.XAPIWrapper.sendState(attemptIri, agent, constants.attemptStateIri, null, state, ADL.XAPIWrapper.hash(asStr));
            }, 0);
        }
    }

    /*******************************************************************************
     **
     ** This function is used to route set values to the appropriate functions
     **
     *******************************************************************************/
    var saveDataValue = function (name, value, preValue) {
        var isInteraction = name.indexOf("cmi.interactions") > -1;
        var attemptIri = window.localStorage[config.activityId];
        var agent = getAgent();

        if (isInteraction) {
            setInteraction(name, value);
        } else {
            // Handle only non-array scorm data model elements  
            switch (name) {
                case scormVersionConfig.scoreScaledElement:
                    setScore(value);
                    break;
                case scormVersionConfig.completionElement:
                    setComplete(value);
                    break;
                case scormVersionConfig.successElement:
                    setSuccess(value);
                    break;
                case scormVersionConfig.exitElement:
                    exitSetToSuspend = (value == "suspend");
                    break;
                case scormVersionConfig.locationElement:
                    if (preValue === value) { break; }
                    if (registration) {
                        setTimeout(() => {
                            ADL.XAPIWrapper.sendState(config.activityId, agent, 'bookmark', registration, value);
                        }, 0);
                    }
                    break;
                case scormVersionConfig.suspendDataElement:
                    if (preValue === value) { break; }
                    if (registration) {
                        setTimeout(() => {
                            ADL.XAPIWrapper.sendState(config.activityId, agent, 'suspend_data', registration, value);
                        }, 0);
                    }
                    break;
                default:
                    break;
            }
        }
    }
    /*******************************************************************************
     **
     ** This function/vars is used to handle the interaction type
     **
     *******************************************************************************/
    var setInteraction = function (name, value) {
        // key for interactions in local storage is scoped to an attempt
        var interactionsKey = window.localStorage[config.activityId] + "_interactions";

        // get the interactions from local storage
        var cachedInteractionsStr = window.localStorage.getItem(interactionsKey);
        var cachedInteractions = [];
        if (cachedInteractions != null) {
            // get as JSON object array
            var cachedInteractions = JSON.parse(cachedInteractionsStr);
        }

        // figure out what the set value was in the SCORM call
        elementArray = name.split(".");
        var intIndex = elementArray[2];
        var subElement = elementArray[3];

        if (subElement == "id") {
            // its a new interaction.  Set it in local storage
            var newInteraction = {
                index: intIndex,
                id: value,
                type: "",
                learner_response: "",
                result: "",
                description: ""
            };

            if (cachedInteractions != null) {
                // this is not the first interaction set
                cachedInteractions.push(newInteraction);

                // push to local storage
                window.localStorage.setItem(interactionsKey, JSON.stringify(cachedInteractions));
            } else {
                // this is the first interaction set
                window.localStorage.setItem(interactionsKey, JSON.stringify([newInteraction]));
            }
        } else if (subElement == "type") {
            // find interaction with the same index and set type in JSON array
            for (var i = 0; i < cachedInteractions.length; i++) {
                if (cachedInteractions[i].index == intIndex) {
                    // found matching index so update this object's type
                    cachedInteractions[i].type = value;

                    // update local storage
                    window.localStorage.setItem(interactionsKey, JSON.stringify(cachedInteractions));

                    break;
                }
            }
        } else if (subElement == "description") {
            // find interaction with the same index and set type in JSON array
            for (var i = 0; i < cachedInteractions.length; i++) {
                if (cachedInteractions[i].index == intIndex) {
                    // found matching index so update this object's type
                    cachedInteractions[i].description = value;

                    // update local storage
                    window.localStorage.setItem(interactionsKey, JSON.stringify(cachedInteractions));

                    break;
                }
            }
        } else if (subElement == "learner_response" || subElement == "student_response") {
            // find interaction with the same index and set type in JSON array
            for (var i = 0; i < cachedInteractions.length; i++) {
                if (cachedInteractions[i].index == intIndex) {
                    // found matching index so update this object's type
                    cachedInteractions[i].learner_response = value;

                    // update local storage
                    window.localStorage.setItem(interactionsKey, JSON.stringify(cachedInteractions));

                    // Send xAPI Statement
                    // Note: this implementation
                    var stmt = getInteractionsBaseStatement();
                    stmt.object.id = getInteractionIri(cachedInteractions[i].id);
                    stmt.context.contextActivities.grouping[0].id = window.localStorage[config.activityId];

                    // set the context activity from the manifest/launch_data to group together
                    // for an event
                    //stmt.context.contextActivities.grouping.push(config.groupingContextActivity);

                    // set the learner's response
                    stmt.result.response = cachedInteractions[i].learner_response;

                    // todo: shouldn't assume en-US - implement with default if not specified, or use what was sent
                    if (config.isScorm2004) {
                        stmt.object.definition.description = {
                            "en-US": cachedInteractions[i].description
                        };
                    }

                    // set the specific interaction type
                    stmt.object.definition.interactionType = cachedInteractions[i].type;

                    // get any type specific JSON that an LRS *may* require
                    switch (cachedInteractions[i].type) {
                        case "choice":
                            stmt.object.definition.choices = [];
                            break;
                        case "likert":
                            stmt.object.definition.scale = [];
                            break;
                        case "matching":
                            stmt.object.definition.source = [];
                            stmt.object.definition.target = [];
                            break;
                        case "performance":
                            stmt.object.definition.steps = [];
                            break;
                        case "sequencing":
                            stmt.object.definition.choices = [];
                            break;
                        default:
                            break;
                    }

                    // todo: make the subelement that you send stmt on configurable
                    // send statement
                    setTimeout(() => {
                        var response = ADL.XAPIWrapper.sendStatement(stmt);
                    }, 0);

                    // remove interaction from local storage array so its not processed again
                    cachedInteractions.splice(i, 1);
                }
            }
        }

    }

    /*******************************************************************************
     **
     ** This function is used to get an interaction iri
     **
     *******************************************************************************/
    var getInteractionIri = function (interactionId) {
        return config.activityId + "/interactions/" + encodeURIComponent(interactionId);
    }

    /*******************************************************************************
     **
     ** This function is used to set a scaled score
     **
     *******************************************************************************/
    var setScore = function (value, id, description, res) {
        // For scorm 1.2, must divide raw by 100
        var score = (config.isScorm2004) ? parseFloat(value) : parseFloat(value) / 100;

        var stmt = getBaseStatement();
        stmt.verb = ADL.verbs.scored;
        stmt.context.contextActivities.grouping[0].id = window.localStorage[config.activityId];
        stmt.object.id = stmt.object.id + "/" + (id || "Assessment");
        stmt.object.definition.description = {
            "en-US": description || "Finish the assessment"
        };
        // set the context activity from the manifest/launch_data to group together
        // for an event
        // stmt.context.contextActivities.grouping.push(config.groupingContextActivity);

        // todo: add error handling if value is not a valid scaled score
        stmt.result = {
            score: {
                scaled: score
            }
        };

        if (!!res) {
            stmt.result.response = res;
        }

        setTimeout(() => {
            var response = ADL.XAPIWrapper.sendStatement(stmt);
        }, 0);
    }

    /*******************************************************************************
     **
     ** This function is used to set experience statement
     **
     *******************************************************************************/

    var setExperienceBlockStatement = function (title, id) {

        var stmt = getBaseStatement();

        stmt.verb = ADL.verbs.experienced;
        stmt.context.contextActivities.grouping[0].id = window.localStorage[config.activityId];
        stmt.object.id = stmt.object.id + '/' + id;
        stmt.object.definition.description = {
            "en-US": "Opened: " + title
        };

        setTimeout(() => {
            var response = ADL.XAPIWrapper.sendStatement(stmt);
        }, 0);
    }

    /*******************************************************************************
     **
     ** This function is used to complete an activity
     **
     *******************************************************************************/
    var setComplete = function (value) {
        if (value == "completed") {
            sendSimpleStatement(ADL.verbs.completed);
        } else if (value == "passed" || value == "failed") {
            sendSimpleStatement(ADL.verbs[value]);
        } else {
            sendSimpleStatement(ADL.verbs.attempted);
        }
    }

    /*******************************************************************************
     **
     ** This function is used to set pass/failed on an activity
     **
     *******************************************************************************/
    var setSuccess = function (value) {
        // if SCORM 1.2, these could be complete/incomplete
        if (value == "passed" || value == "failed")
            sendSimpleStatement(ADL.verbs[value]);
    }

    /*******************************************************************************
     **
     ** This function is used to configure LRS endpoint and other values
     **
     *******************************************************************************/
    //   var setConfig = function(iConfig)
    //   {
    //      config.lrs.endpoint = iConfig.lrs.endpoint;
    //      config.lrs.user = iConfig.lrs.user;
    //      config.lrs.password = iConfig.lrs.password;
    //      config.courseId = iConfig.courseId;
    //      config.lmsHomePage = iConfig.lmsHomePage;
    //      config.isScorm2004 = iConfig.isScorm2004;
    //
    //      // setup SCORM object based on configuration
    //      scormVersionConfig = {
    //         learnerIdElement: (config.isScorm2004) ? "cmi.learner_id" : "cmi.core.student_id",
    //         entryElement: ((config.isScorm2004 == true) ? "cmi.entry" : "cmi.core.entry"),
    //         exitElement: (config.isScorm2004) ? "cmi.exit" : "cmi.core.exit",
    //         successElement: (config.isScorm2004) ? "cmi.success_status" : "cmi.core.lesson_status",
    //         completionElement: (config.isScorm2004) ? "cmi.completion_status" : "cmi.core.lesson_status",
    //         scoreRawElement: (config.isScorm2004) ? "cmi.score.raw" : "cmi.core.score.raw",
    //         scoreMinElement: (config.isScorm2004) ? "cmi.score.min" : "cmi.core.score.min",
    //         scoreMaxElement: (config.isScorm2004) ? "cmi.score.max" : "cmi.core.score.max",
    //         scoreScaledElement: (config.isScorm2004) ? "cmi.score.scaled" : "cmi.core.score.raw",
    //         languageElement: (config.isScorm2004) ? "cmi.learner_preference.language" : "cmi.student_preference.language",
    //         audioLevelElement: (config.isScorm2004) ? "cmi.learner_preference.audio_level" : "cmi.student_preference.audio",
    //         deliverySpeedElement: (config.isScorm2004) ? "cmi.learner_preference.delivery_speed" : "cmi.student_preference.speed",
    //         audioCaptioningElement: (config.isScorm2004) ? "cmi.learner_preference.audio_captioning" : "cmi.student_preference.text",
    //         completionThresholdElement: (config.isScorm2004) ? "cmi.completion_threshold" : "",
    //         launchDataElement: "cmi.launch_data",
    //         maxTimeAllowedElement: (config.isScorm2004) ? "cmi.max_time_allowed" : "cmi.student_data.max_time_allowed",
    //         scaledPassingScoreElement: (config.isScorm2004) ? "cmi.scaled_passing_score" : "cmi.student_data.mastery_score",
    //         timeLimitActionElement: (config.isScorm2004) ? "cmi.time_limit_action" : "cmi.student_data.time_limit_action",
    //         locationElement: (config.isScorm2004) ? "cmi.location" : "cmi.core.lesson_location",
    //         creditElement: (config.isScorm2004) ? "cmi.credit" : "cmi.core.credit",
    //         modeElement: (config.isScorm2004) ? "cmi.mode" : "cmi.core.lesson_mode",
    //         suspendDataElement: "cmi.suspend_data",
    //         totalTimeElement: (config.isScorm2004) ? "cmi.total_time" : "cmi.core.total_time"
    //      }
    //
    //   }

    /*******************************************************************************
     **
     ** This function is used to configure LRS endpoint and other values
     **
     *******************************************************************************/
    var configureXAPIData = function () {
        // setup SCORM object based on configuration
        scormVersionConfig = {
            learnerIdElement: (config.isScorm2004) ? "cmi.learner_id" : "cmi.core.student_id",
            entryElement: ((config.isScorm2004 == true) ? "cmi.entry" : "cmi.core.entry"),
            exitElement: (config.isScorm2004) ? "cmi.exit" : "cmi.core.exit",
            successElement: (config.isScorm2004) ? "cmi.success_status" : "cmi.core.lesson_status",
            completionElement: (config.isScorm2004) ? "cmi.completion_status" : "cmi.core.lesson_status",
            scoreRawElement: (config.isScorm2004) ? "cmi.score.raw" : "cmi.core.score.raw",
            scoreMinElement: (config.isScorm2004) ? "cmi.score.min" : "cmi.core.score.min",
            scoreMaxElement: (config.isScorm2004) ? "cmi.score.max" : "cmi.core.score.max",
            scoreScaledElement: (config.isScorm2004) ? "cmi.score.scaled" : "cmi.core.score.raw",
            languageElement: (config.isScorm2004) ? "cmi.learner_preference.language" : "cmi.student_preference.language",
            audioLevelElement: (config.isScorm2004) ? "cmi.learner_preference.audio_level" : "cmi.student_preference.audio",
            deliverySpeedElement: (config.isScorm2004) ? "cmi.learner_preference.delivery_speed" : "cmi.student_preference.speed",
            audioCaptioningElement: (config.isScorm2004) ? "cmi.learner_preference.audio_captioning" : "cmi.student_preference.text",
            completionThresholdElement: (config.isScorm2004) ? "cmi.completion_threshold" : "",
            launchDataElement: "cmi.launch_data",
            maxTimeAllowedElement: (config.isScorm2004) ? "cmi.max_time_allowed" : "cmi.student_data.max_time_allowed",
            scaledPassingScoreElement: (config.isScorm2004) ? "cmi.scaled_passing_score" : "cmi.student_data.mastery_score",
            timeLimitActionElement: (config.isScorm2004) ? "cmi.time_limit_action" : "cmi.student_data.time_limit_action",
            locationElement: (config.isScorm2004) ? "cmi.location" : "cmi.core.lesson_location",
            creditElement: (config.isScorm2004) ? "cmi.credit" : "cmi.core.credit",
            modeElement: (config.isScorm2004) ? "cmi.mode" : "cmi.core.lesson_mode",
            suspendDataElement: "cmi.suspend_data",
            totalTimeElement: (config.isScorm2004) ? "cmi.total_time" : "cmi.core.total_time"
        }

    }

    /*******************************************************************************
     **
     ** This function is used to get the attempt context activity (grouping) id
     **
     *******************************************************************************/
    var configureAttemptContextActivityID = function (cmiEntryValue) {
        // window.localStorage[config.activityId] uses activity id to return the most recent
        // attempt
        if (cmiEntryValue == "resume") {
            if (!window.localStorage[config.activityId]) {
                window.localStorage[config.activityId] = config.activityId;
            }

            // send a resume statement
            //resumeAttempt();

        } else {
            window.localStorage[config.activityId] = config.activityId;

            // update the activity state with the new attempt IRI
            setActivityState();
        }
    }

    /*******************************************************************************
     **
     ** Sends same basic statement with varying verbs 
     **
     *******************************************************************************/
    var sendSimpleStatement = function (verb) {
        var stmt = getBaseStatement();
        stmt.verb = verb;
        stmt.context.contextActivities.grouping[0].id = window.localStorage[config.activityId];

        // set the context activity from the manifest/launch_data to group together
        // for an event
        // stmt.context.contextActivities.grouping.push(config.groupingContextActivity);

        setTimeout(() => {
            var response = ADL.XAPIWrapper.sendStatement(stmt);
        }, 0);
    }


    /*******************************************************************************
     **
     ** This function is used to (most likely) get a unique guid to identify 
     ** an attempt
     **
     *******************************************************************************/
    var generateUUID = function () {
        var d = new Date().getTime();

        var uuid = 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function (c) {
            var r = (d + Math.random() * 16) % 16 | 0;
            d = Math.floor(d / 16);
            return (c == 'x' ? r : (r & 0x7 | 0x8)).toString(16);
        });

        return uuid;
    }

    function message(str) {
        if (_debug) {
            output.log(str);
        }
    }

    return {
        //newSetConfig: newSetConfig,
        //setConfig:setConfig,
        initializeAttempt: initializeAttempt,
        resumeAttempt: resumeAttempt,
        suspendAttempt: suspendAttempt,
        terminateAttempt: terminateAttempt,
        saveDataValue: saveDataValue,
        setScore: setScore,
        setComplete: setComplete,
        setSuccess: setSuccess,
        setExperienceBlockStatement: setExperienceBlockStatement,
        configureLRS: configureLRS
    }

    // 
}(); // end xapi object
