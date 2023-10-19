from jira import JIRA, JIRAError
from jira.resources import *
from jira.client import JIRA

import os
import sys
import html_text
import requests
import re
import json
import csv
import time
import random
from getpass import getpass
from PySide6.QtCore import QObject, Slot
from PySide6.QtGui import QGuiApplication
from PySide6.QtQml import QQmlApplicationEngine, QmlElement

QML_IMPORT_NAME = "io.qt.textproperties"
QML_IMPORT_MAJOR_VERSION = 1

@QmlElement
class Bridge(QObject):
    # 로그인
    @Slot(str, str, str, result=bool)
    def login(self, url, username, password):
        try:
            global jira
            jira = JIRA(server=url, basic_auth=(username, password))
            return 1
        except JIRAError as err:
            print("=" * 100)
            print("    {} **".format(err))
            print("=" * 100)
            return 0
    
    # 샘플 테스트 - 이슈 보기 (10개만)
    @Slot(str)
    def get_issues_sample(self, query):
        print("이슈 보기 (10개만)")
        print("다음 쿼리를 실행하여 나온 결과 중 최초 10개만 콘솔에 보여 드립니다.")
        print(query)
        issues = jira.search_issues(query, startAt=1, maxResults=10)
        for issue in issues:
            try:
                print("Parent: %s" % issue.fields.parent)
            except AttributeError as err:
                print("Parent: -")
            print("  Key@: %s" % issue.key)
            print("  Project: %s" % issue.fields.project)
            print("  Summary: %s" % issue.fields.summary)
            print("Details")
            print("  Type: %s" % issue.fields.issuetype)
            print("  Priority: %s" % issue.fields.priority)
            for component in issue.fields.components:
                print("  Components: %s" % component.name)
            print("  Labels: %s" % issue.fields.labels)
            print("  HMC프로젝트: %s" % issue.get_field('customfield_43801'))   # ID: 'customfield_43801', name: 'HMC프로젝트' (커스텀 필드의 값은 jira.fields() 함수로 찾을 수 있음)
            print("  Status@: %s" % issue.fields.status)
            print("  Resolution@: %s" % issue.fields.resolution)
            print("People")
            print("  Assignee: %s" % issue.fields.assignee)
            print("  Reporter@: %s" % issue.fields.reporter)
            watchers = jira.watchers(issue)
            for watcher in watchers.watchers:
                print("  Watcher: %s" % watcher)
            print("Dates")
            print("  Due: %s" % issue.fields.duedate)
            print("  Created@: %s" % issue.fields.created)
            print("Description: %s" % issue.fields.description)
            print("\n")
    
    # 샘플 테스트 - 이슈 생성 (1개만)
    @Slot()
    def make_an_issue_sample(self):
        # dictionary 자료형
        # components = []
        # for component in issue.fields.components:
        #     components.append({'name': str(component.name)})
        issue_dict = {
            'project': 'HKMCCLUHUD',
            'summary': 'test issue mmm',
            'issuetype': {'name': 'Task'},
            'priority': {'name': 'P2'},
            'components': [{'name': 'ES94111-01'}],
            'labels': ['ccIC24_CLU_WBS'],
            'customfield_43801': {'value': 'ccIC24'},       # HMC프로젝트
            'customfield_10104': {'value': 'Comment'},      # Severity
            'assignee': {'name': 'soonbum.jeong'},
            'duedate': '2023-08-30',
            'description': '설명 텍스트입니다',
            #'parent': {'key': issue.key},       # issuetype == Sub-task의 경우 parent를 지정해야 함
            'versions': [{'name': 'N/A'}],      # Affects Versions
            'fixVersions': [{'name': 'N/A'}],   # Fix Versions
        }
        print("다음 정보를 기반으로 이슈 하나를 생성해 보겠습니다.")
        print('입력한 정보: ', issue_dict)
        new_issue = jira.create_issue(issue_dict)
        # Watcher 정보는 따로 추가해야 함
        jira.add_watcher(new_issue, jira._get_user_id('soonbum.jeong'))
        #jira.remove_watcher(new_issue, jira._get_user_id('soonbum.jeong'))
        print('생성된 이슈: ', new_issue)

    # 이슈 관리 - 이슈 수집하기 (CSV)
    @Slot(str)
    def collect_all_issues(self, query):
        # 모든 이슈를 가져옴
        issues = jira.search_issues(query, maxResults=0)

        # csv 파일 헤더 작성
        filename_to_write = os.path.dirname(__file__) + '\\[HKMCCLUHUD][ccIC24] issues.csv'
        with open(filename_to_write, 'w', encoding='euc-kr', newline='') as data_to_write:
            csvwriter = csv.writer(data_to_write, delimiter=',')
            csvwriter.writerow(['update', 'key*', 'project*', 'summary', 'issuetype', 'priority', 'components', 'labels', 'HMC프로젝트', 'status*', 'resolution*', 'assignee', 'reporter*', 'watchers', 'duedate', 'created*', 'description'])

        # 이슈를 csv 파일에 기록하기
        for issue in issues:
            components = []
            for component in issue.fields.components:
                components.append(component.name)

            hmcProject = issue.get_field('customfield_43801')

            watcher_list = []
            watchers = jira.watchers(issue)
            for watcher in watchers.watchers:
                watcher_list.append(str(watcher))

            with open(filename_to_write, 'a', encoding='euc-kr', newline='') as data_to_write:
                csvwriter = csv.writer(data_to_write, delimiter=',')
                try:
                    csvwriter.writerow(['', issue.key, issue.fields.project, issue.fields.summary, issue.fields.issuetype, issue.fields.priority, components, issue.fields.labels, hmcProject, issue.fields.status, issue.fields.resolution, issue.fields.assignee, issue.fields.reporter, watcher_list, issue.fields.duedate, issue.fields.created, issue.fields.description])
                except UnicodeEncodeError as err:
                    print("%s" % issue.key)
                    print("    {} **".format(err))
                    description_str = str(issue.fields.description)
                    description = description_str.encode(encoding = "euc-kr", errors = "ignore")
                    csvwriter.writerow(['', issue.key, issue.fields.project, issue.fields.summary, issue.fields.issuetype, issue.fields.priority, components, issue.fields.labels, hmcProject, issue.fields.status, issue.fields.resolution, issue.fields.assignee, issue.fields.reporter, watcher_list, issue.fields.duedate, issue.fields.created, description])

    # 이슈 관리 - 이슈 업데이트 (CSV)
    @Slot()
    def update_all_issues(self):
        # csv 파일 가져오기
        filename_to_read = os.path.dirname(__file__) + '\\[HKMCCLUHUD][ccIC24] issues.csv'
        with open(filename_to_read, 'r', encoding='euc-kr', newline='') as data_to_read:
            for line in csv.reader(data_to_read):
                if(line[0] != 'update'):    # 머리말이 아닐 경우에만 다음 절차 진행
                    if(line[0] != ''):      # 업데이트 flag가 입력되어 있을 경우
                        # 키를 이용하여 이슈 가져오기
                        issue = jira.issue(line[1])

                        components_str = json.loads(line[6].replace("'", "\""))
                        labels_str = json.loads(line[7].replace("'", "\""))
                        components = []
                        for component in components_str:
                            components.append({'name': component})
                        labels = []
                        for label in labels_str:
                            labels.append(label)
                        issue_dict = {
                            'summary': line[3],
                            'issuetype': {'name': line[4]},
                            'priority': {'name': line[5]},
                            'components': components,
                            'labels': labels,
                            'customfield_43801': {'value': line[8]},       # HMC프로젝트
                            'assignee': {'name': line[11].split(' ')[-1]},
                            'duedate': line[14],
                            'description': line[16],
                        }
                        issue.update(notify = True, fields = issue_dict)
                        
                        # Watcher 추가
                        watchers_str = json.loads(line[13].replace("'", "\""))
                        for watcher in watchers_str:
                            jira.add_watcher(issue, jira._get_user_id(str(watcher).split(' ')[-1]))

    # 커스텀 기능 - 이슈 복사하고 제목 바꾸기
    @Slot(str)
    def custom_issue_cloning_and_renaming(self, query):
        # 프로젝트명(HKMCCLUHUD) | (ccIC24), 카테고리 (WBS3)
        # 기존 제목 (예: Analysis), 새로운 제목 (예: SyRS), Due Date (예: 2023-08-31) 입력 받기
        # 조건: 다 복사하고, 기존 이슈의 parent를 가져와서 새 이슈의 parent로 연결할 것
        old_title = input("이전 제목(예: Analysis): ")
        new_title = input("새로운 제목(예: SyRS): ")
        due_date = input("마감기한(예: 2023-08-31): ")
        delay_seconds = input("업데이트 딜레이 초(120): ")

        # 제목이 old_title인 이슈를 가져옴
        append_query = " AND summary ~ %s" % old_title
        query_old_title = query + append_query
        issues = jira.search_issues(query_old_title, maxResults=0)

        seq = 1
        for issue in issues:
            print("\n[%d] %s: %s" % (seq, issue.key, issue.fields.summary))

            components = []
            for component in issue.fields.components:
                components.append({'name': str(component.name)})
            labels = []
            for label in issue.fields.labels:
                labels.append(label)
            issue_dict = {
                'project': str(issue.fields.project),
                'summary': str(issue.fields.summary).replace(old_title, new_title),
                'issuetype': {'name': str(issue.fields.issuetype)},
                'priority': {'name': str(issue.fields.priority)},
                'components': components,
                'labels': labels,
                'customfield_43801': {'value': str(issue.get_field('customfield_43801'))},       # HMC프로젝트
                'assignee': {'name': str(issue.fields.assignee).split(' ')[-1]},     # 마지막 영문 이름만 사용하기
                'duedate': due_date,
                'description': str(issue.fields.description),            
                'parent': {'key': str(issue.fields.parent.key)},       # Sub-task의 경우 parent를 지정해야 함
                'versions': [{'name': 'None'}],     # Affects Versions
                'fixVersions': [{'name': 'None'}],  # Fix Versions
            }
            print('  입력한 정보: ', issue_dict)

            # 만약 이미 생성된 제목이 있는지 확인할 것
            append_query = " AND summary ~ %s" % new_title
            query_new_title = query + append_query
            another_query = query_new_title
            another_issues = jira.search_issues(another_query, maxResults=0)
            find_already_made_issue = False
            for another_issue in another_issues:
                # another_issue 중에는 "...]Analysis"도 있고 "...] Analysis"도 있을 수 있다.
                comparison_str_1 = str(another_issue.fields.summary)
                comparison_str_2 = ''
                if(comparison_str_1.find('] ' + new_title) != -1):
                    comparison_str_2 = str(comparison_str_1.replace(('] ' + new_title), (']' + new_title)))
                else:
                    comparison_str_2 = str(comparison_str_1.replace((']' + new_title), ('] ' + new_title)))
                # 제목 맨 앞에 공백 문자가 들어가는 경우도 있을 수 있다.
                comparison_str_3 = ' ' + comparison_str_1
                comparison_str_4 = ' ' + comparison_str_2

                if(str(issue.fields.summary.replace(old_title, new_title)) == comparison_str_1):
                    find_already_made_issue = True
                if(str(issue.fields.summary.replace(old_title, new_title)) == comparison_str_2):
                    find_already_made_issue = True
                if(str(issue.fields.summary.replace(old_title, new_title)) == comparison_str_3):
                    find_already_made_issue = True
                if(str(issue.fields.summary.replace(old_title, new_title)) == comparison_str_4):
                    find_already_made_issue = True

            # 아직 이슈를 생성하지 않았다면 만들 것
            if(find_already_made_issue == False):
                new_issue = jira.create_issue(issue_dict)
                # Watcher 정보는 따로 추가해야 함
                watchers = jira.watchers(issue)
                for watcher in watchers.watchers:
                    jira.add_watcher(new_issue, jira._get_user_id(watcher))
                print('[%d] 생성된 이슈: %s' % (seq, new_issue))
                time.sleep(int(delay_seconds))  # delay_seconds초 대기
            else:
                print('[%d] 이미 생성된 이슈: %s' % (seq, issue.key))

            seq = seq + 1

        print("이슈 복사하고 제목 바꾸기: 작업을 완료했습니다.")

    # 커스텀 기능 - 특정 assignee/watcher인 이슈에 watcher 추가하기
    @Slot(str)
    def add_watchers_of_specific_person(self, query):
        # 예: jaeseon.yoon이 assignee 또는 watcher일 경우, nayoung.choi, hyomin.jun을 watcher로 추가하기
        name_to_find = input("찾고 싶은 assignee/watcher 이름을 입력하세요(예: jaeseon.yoon): ")
        watcher_list = []
        nPersons = input("추가하고 싶은 watcher 수를 입력하세요(예: 2): ")
        print("예) 이름을 순서대로 입력해 보세요... nayoung.choi, hyomin.jun")
        for i in range(int(nPersons)):
            name_to_add = input("%d번째 이름을 입력하세요: " % (i+1))
            watcher_list.append(name_to_add)
        delay_seconds = input("업데이트 딜레이 초(30): ")

        issues = jira.search_issues(query, maxResults=0)
        find_person = False
        seq = 1
        for issue in issues:
            # name_to_find에 입력된 이름이 이슈의 assignee 또는 watcher에 있는지 찾아볼 것
            find_person = False
            if(str(issue.fields.assignee).split(' ')[-1] == name_to_find):
                find_person = True
            watchers = jira.watchers(issue)
            for watcher in watchers.watchers:
                if(str(watcher).split(' ')[-1] == name_to_find):
                    find_person = True

            # 이름을 찾았으면 추가하고 싶었던 watcher를 추가함
            if(find_person == True):
                for watcher_name in watcher_list:
                    jira.add_watcher(issue, watcher_name)
                print("[%d][%s : %s] 특정 assignee/watcher인 이슈에 watcher 추가하기: 작업 완료" % (seq, issue.key, issue.fields.summary))

            time.sleep(int(delay_seconds))  # delay_seconds초 대기
            seq = seq + 1

        print("특정 assignee/watcher인 이슈에 watcher 추가하기: 작업을 완료했습니다.")

    # 커스텀 기능 - 특정 watcher를 모든 이슈에서 제거하기
    @Slot(str)
    def del_watcher_from_all_issues(self, query):
        name_to_delete = input("지우고 싶은 watcher 이름을 입력하세요(예: jimin91.song): ")

        issues = jira.search_issues(query, maxResults=0)
        find_person = False
        seq = 1
        for issue in issues:
            # name_to_delete에 입력된 이름이 이슈의 watcher에 있는지 찾아볼 것
            find_person = False
            watchers = jira.watchers(issue)
            for watcher in watchers.watchers:
                if(str(watcher).split(' ')[-1] == name_to_delete):
                    find_person = True

            # 이름을 찾았으면 지우고 싶었던 watcher를 제거함
            if(find_person == True):
                jira.remove_watcher(issue, jira._get_user_id(name_to_delete))
                print("[%d][%s : %s] 특정 watcher를 모든 이슈에서 제거하기: 작업 완료" % (seq, issue.key, issue.fields.summary))
            
            seq = seq + 1
        
        print("특정 watcher를 모든 이슈에서 제거하기: 작업을 완료했습니다.")

QML = """
import QtQuick
import QtQuick.Controls
import QtQuick.Layouts

import io.qt.textproperties 1.0

Window {
    id: main_window
    width: 900
    height: 400
    visible: true
    title: "VLM(JIRA) 자동화 도구"

    Bridge {
        id: bridge
    }
    
    property bool login_success: false
    property string url_to_connect: "http://vlm.lge.com/issue/"
    property string query: ""

    ColumnLayout {
        anchors.fill: parent

        RowLayout {
            Text {
                text: "접속할 URL"
                Layout.margins: 4
            }
            TextField {
                id: url_string
                text: url_to_connect
                Layout.margins: 4
                implicitWidth: 200
            }
            Text {
                text: "ID"
                Layout.margins: 4
            }
            TextField {
                id: id_string
                placeholderText: qsTr("ID를 입력하세요")
                Layout.margins: 4
                implicitWidth: 150
            }
            Text {
                text: "Password"
                Layout.margins: 4
            }
            TextField {
                id: password_string
                echoMode: TextInput.Password
                placeholderText: qsTr("암호를 입력하세요")
                Layout.margins: 4
                implicitWidth: 150
            }
            Button {
                id: login_button
                text: "로그인"
                Layout.margins: 4
                enabled: true
                onClicked: {
                    if(bridge.login(url_string.text, id_string.text, password_string.text) == 1) {
                        login_success = true
                        login_state.text = "<b>로그인 성공</b>"
                        query = "project in (" + project_name_string.text + ")"
                        if(bKeyword1.checked == true)   query = query + " AND summary ~ " + keyword1_string.text
                        if(bKeyword2.checked == true)   query = query + " AND summary ~ " + keyword2_string.text
                        if(bKeyword3.checked == true)   query = query + " AND summary ~ " + keyword3_string.text
                        jql_query_string.text = query
                    }
                    else {
                        login_success = false
                        login_state.text = "<b>로그인 실패 (ID 또는 패스워드를 확인하십시오)</b>"
                    }
                }
            }
        }
        RowLayout {
            Text {
                id: login_state
                text: "<b>먼저 로그인을 하십시오.</b>"
                Layout.margins: 4
            }
        }
        RowLayout {
            Text {
                text: ">> 조건"
                Layout.margins: 4
            }
        }
        RowLayout {
            CheckBox {
                id: bProject
                checked: true
                enabled: false
                text: qsTr("프로젝트")
            }
            TextField {
                id: project_name_string
                text: "HKMCCLUHUD"
                Layout.margins: 4
            }
            CheckBox {
                id: bKeyword1
                checked: true
                text: qsTr("키워드1")
                onToggled: {
                    if(login_success == true) {
                        query = "project in (" + project_name_string.text + ")"
                        if(bKeyword1.checked == true)   query = query + " AND summary ~ " + keyword1_string.text
                        if(bKeyword2.checked == true)   query = query + " AND summary ~ " + keyword2_string.text
                        if(bKeyword3.checked == true)   query = query + " AND summary ~ " + keyword3_string.text
                        jql_query_string.text = query
                    }
                }
            }
            TextField {
                id: keyword1_string
                text: "ccIC24"
                Layout.margins: 4
            }
            CheckBox {
                id: bKeyword2
                checked: true
                text: qsTr("키워드2")
                onToggled: {
                    if(login_success == true) {
                        query = "project in (" + project_name_string.text + ")"
                        if(bKeyword1.checked == true)   query = query + " AND summary ~ " + keyword1_string.text
                        if(bKeyword2.checked == true)   query = query + " AND summary ~ " + keyword2_string.text
                        if(bKeyword3.checked == true)   query = query + " AND summary ~ " + keyword3_string.text
                        jql_query_string.text = query
                    }
                }
            }
            TextField {
                id: keyword2_string
                text: "WBS3"
                Layout.margins: 4
            }
            CheckBox {
                id: bKeyword3
                checked: false
                text: qsTr("키워드3")
                onToggled: {
                    if(login_success == true) {
                        query = "project in (" + project_name_string.text + ")"
                        if(bKeyword1.checked == true)   query = query + " AND summary ~ " + keyword1_string.text
                        if(bKeyword2.checked == true)   query = query + " AND summary ~ " + keyword2_string.text
                        if(bKeyword3.checked == true)   query = query + " AND summary ~ " + keyword3_string.text
                        jql_query_string.text = query
                    }
                }
            }
            TextField {
                id: keyword3_string
                text: ""
                Layout.margins: 4
            }
        }
        RowLayout {
            Text {
                text: "생성된 쿼리문"
                Layout.margins: 4
            }
            TextField {
                id: jql_query_string
                text: ""
                Layout.margins: 4
                implicitWidth: 700
            }
        }
        RowLayout {
            Text {
                text: ">> 기능"
                Layout.margins: 4
            }
        }
        RowLayout {
            Text {
                text: "샘플 테스트"
                Layout.margins: 4
            }
            Button {
                id: button_get_issues_sample
                text: "이슈 보기 (10개만)"
                Layout.margins: 4
                onClicked: {
                    bridge.get_issues_sample(jql_query_string.text)
                }
            }
            Button {
                id: button_make_an_issue_sample
                text: "이슈 생성 (1개만)"
                Layout.margins: 4
                onClicked: {
                    bridge.make_an_issue_sample()
                }
            }
        }
        RowLayout {
            Text {
                text: "이슈 관리"
                Layout.margins: 4
            }
            Button {
                id: button_collect_all_issues
                text: "이슈 수집하기 (CSV)"
                Layout.margins: 4
                onClicked: {
                    bridge.collect_all_issues(jql_query_string.text)
                }
            }
            Button {
                id: button_update_all_issues
                text: "이슈 업데이트 (CSV)"
                Layout.margins: 4
                onClicked: {
                    bridge.update_all_issues()
                }
            }
        }
        RowLayout {
            Text {
                text: "커스텀 기능"
                Layout.margins: 4
            }
            Button {
                id: button_custom_issue_cloning_and_renaming
                text: "이슈 복사하고 제목 바꾸기"
                Layout.margins: 4
                onClicked: {
                    bridge.custom_issue_cloning_and_renaming(jql_query_string.text)
                }
            }
            Button {
                id: button_add_watchers_of_specific_person
                text: "특정 assignee/watcher인 이슈에 watcher 추가하기"
                Layout.margins: 4
                onClicked: {
                    bridge.add_watchers_of_specific_person(jql_query_string.text)
                }
            }
            Button {
                id: button_del_watcher_from_all_issues
                text: "특정 watcher를 모든 이슈에서 제거하기"
                Layout.margins: 4
                onClicked: {
                    bridge.del_watcher_from_all_issues(jql_query_string.text)
                }
            }
        }
    }
}
"""

if __name__ == "__main__":
    app = QGuiApplication(sys.argv)
    engine = QQmlApplicationEngine()
    engine.loadData(QML.encode('utf-8'))
    if not engine.rootObjects():
        sys.exit(-1)
    exit_code = app.exec()
    del engine
    sys.exit(exit_code)
