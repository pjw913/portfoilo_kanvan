<template>
  <div class="app">
    <!-- 네비게이션 헤더 -->
    <div class="nav-header">
      <div class="nav-tabs">
        <button 
          class="nav-tab" 
          :class="{ active: currentView === 'board' }" 
          @click="currentView = 'board'">
          <i class="fas fa-clipboard-list"></i> 칸반 보드
        </button>
        <button 
          class="nav-tab" 
          :class="{ active: currentView === 'members' }" 
          @click="currentView = 'members'">
          <i class="fas fa-users"></i> 담당자 관리
        </button>
        <button 
          class="nav-tab" 
          :class="{ active: currentView === 'guide' }" 
          @click="currentView = 'guide'">
          <i class="fas fa-info-circle"></i> 사용 가이드
        </button>
      </div>
    </div>

    <!-- 칸반 보드 뷰 -->
    <div v-if="currentView === 'board'" class="board-view">
      <div class="header">
        <h1><i class="fas fa-clipboard-list"></i> 칸반 보드</h1>
        <p>효율적인 프로젝트 관리를 위한 드래그 앤 드롭 태스크 보드</p>
        <div class="stats">
          <div class="stat-item">
            <i class="fas fa-tasks"></i> 총 태스크: {{ totalTasks }}
          </div>
          <div class="stat-item">
            <i class="fas fa-clock"></i> 진행 중: {{ inProgressTasks }}
          </div>
          <div class="stat-item">
            <i class="fas fa-check-circle"></i> 완료: {{ completedTasks }}
          </div>
          <div class="stat-item">
            <i class="fas fa-chart-line"></i> 진행률: {{ progressPercentage }}%
          </div>
        </div>
        <div class="progress-bar">
          <div class="progress-fill" :style="{ width: progressPercentage + '%' }"></div>
        </div>
      </div>
      
      <div class="board">
        <div class="column" v-for="column in columns" :key="column.id">
          <div class="column-header">
            <div class="column-title">
              <i :class="column.icon"></i>
              {{ column.title }}
              <span class="task-count">{{ column.tasks.length }}</span>
            </div>
            <button class="add-task-btn" @click="openTaskModal(column.id)">
              <i class="fas fa-plus"></i> 추가
            </button>
          </div>
          
          <div class="column-description">
            {{ column.description }}
          </div>
          
          <div class="tasks-container" 
               :class="{ 'drag-over': dragOverColumn === column.id }"
               @dragover.prevent="handleDragOver(column.id)"
               @dragleave="handleDragLeave"
               @drop="handleDrop($event, column.id)">
            <div v-if="column.tasks.length === 0" class="empty-state">
              <i class="fas fa-inbox"></i><br>
              태스크가 없습니다
            </div>
            <div v-else
                 v-for="task in column.tasks" 
                 :key="task.id"
                 class="task-card"
                 :class="{ 'dragging': draggingTask === task.id }"
                 draggable="true"
                 @dragstart="handleDragStart($event, task)"
                 @dragend="handleDragEnd">
              <div class="task-title">{{ task.title }}</div>
              <div class="task-description" v-if="task.description">{{ task.description }}</div>
              <div class="task-meta">
                <div class="task-assignee" v-if="task.assignee">
                  <i class="fas fa-user"></i> {{ task.assignee }}
                </div>
                <div class="task-priority" :class="'priority-' + task.priority">
                  {{ getPriorityText(task.priority) }}
                </div>
              </div>
              <div class="task-date" v-if="task.dueDate">
                <i class="fas fa-calendar"></i> {{ formatDate(task.dueDate) }}
              </div>
              <div class="task-actions">
                <button class="task-action-btn" @click="editTask(task)" title="수정">
                  <i class="fas fa-edit"></i>
                </button>
                <button class="task-action-btn" @click="deleteTask(task.id)" title="삭제">
                  <i class="fas fa-trash"></i>
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 담당자 관리 뷰 -->
    <div v-if="currentView === 'members'" class="members-view">
      <div class="header">
        <h1><i class="fas fa-users"></i> 담당자 관리</h1>
        <p>프로젝트 팀 멤버를 관리하고 태스크 배정을 위한 담당자를 설정합니다.</p>
      </div>

      <div class="members-container">
        <div class="members-header">
          <h2>팀 멤버 목록</h2>
          <button class="add-member-btn" @click="showAddMemberModal = true">
            <i class="fas fa-user-plus"></i> 멤버 추가
          </button>
        </div>

        <div class="members-grid">
          <div v-for="(member, index) in teamMembers" :key="index" class="member-card">
            <div class="member-avatar">
              <i class="fas fa-user-circle"></i>
            </div>
            <div class="member-info">
              <div class="member-name">{{ member }}</div>
              <div class="member-stats">
                <span class="task-count">할당된 태스크: {{ getAssignedTaskCount(member) }}개</span>
              </div>
            </div>
            <div class="member-actions">
              <button class="member-action-btn edit-btn" @click="editMember(member, index)" title="수정">
                <i class="fas fa-edit"></i>
              </button>
              <button class="member-action-btn delete-btn" @click="deleteMember(index)" title="삭제">
                <i class="fas fa-trash"></i>
              </button>
            </div>
          </div>
        </div>

        <div v-if="teamMembers.length === 0" class="empty-members">
          <i class="fas fa-user-friends"></i>
          <h3>등록된 팀 멤버가 없습니다</h3>
          <p>새로운 팀 멤버를 추가하여 프로젝트를 시작하세요.</p>
        </div>
      </div>
    </div>

    <!-- 사용 가이드 뷰 -->
    <div v-if="currentView === 'guide'" class="guide-view">
      <div class="header">
        <h1><i class="fas fa-info-circle"></i> 칸반 보드 사용 가이드</h1>
        <p>효과적인 프로젝트 관리를 위한 각 컬럼의 역할과 사용법을 안내합니다.</p>
      </div>
      <div class="guide-container">
        <div v-for="guide in guideDetails" :key="guide.id" class="guide-card">
          <div class="guide-header">
            <!-- 아이콘은 id에 따라 다르게 보이고 싶으면 적절히 교체하세요 -->
            <i class="fas fa-clipboard-list"></i>
            <h3>{{ guide.title }}</h3>
          </div>

          <div class="guide-content">
            <p class="guide-description">
              <strong>사용 시점:</strong> {{ guide.when }}
            </p>

            <div class="guide-usage">
              <h4>정확한 역할</h4>
              <p>{{ guide.role }}</p>
            </div>

            <div class="guide-tips">
              <h4>활용 팁</h4>
              <ul>
                <li v-for="(tip, idx) in guide.tips" :key="idx">{{ tip }}</li>
              </ul>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- Task Modal -->
    <div v-if="showTaskModal" class="modal" @click.self="closeTaskModal">
      <div class="modal-content">
        <div class="modal-header">
          <h2 class="modal-title">{{ editingTask ? '태스크 수정' : '새 태스크' }}</h2>
          <button class="close-btn" @click="closeTaskModal">&times;</button>
        </div>
        
        <form @submit.prevent="saveTask">
          <div class="form-group">
            <label class="form-label">제목 *</label>
            <input v-model="taskForm.title" class="form-input" type="text" required>
          </div>
          
          <div class="form-group">
            <label class="form-label">설명</label>
            <textarea v-model="taskForm.description" class="form-textarea" rows="3"></textarea>
          </div>
          
          <div class="form-group">
            <label class="form-label">담당자</label>
            <select v-model="taskForm.assignee" class="form-select">
              <option value="">담당자 선택</option>
              <option v-for="person in teamMembers" :key="person" :value="person">{{ person }}</option>
            </select>
          </div>
          
          <div class="form-group">
            <label class="form-label">우선순위</label>
            <select v-model="taskForm.priority" class="form-select">
              <option value="low">낮음</option>
              <option value="medium">보통</option>
              <option value="high">높음</option>
            </select>
          </div>
          
          <div class="form-group">
            <label class="form-label">마감일</label>
            <input v-model="taskForm.dueDate" class="form-input" type="date">
          </div>
          
          <div class="form-actions">
            <button type="button" class="btn btn-secondary" @click="closeTaskModal">취소</button>
            <button type="submit" class="btn btn-primary">{{ editingTask ? '수정' : '추가' }}</button>
          </div>
        </form>
      </div>
    </div>

    <!-- Add Member Modal -->
    <div v-if="showAddMemberModal" class="modal" @click.self="closeAddMemberModal">
      <div class="modal-content">
        <div class="modal-header">
          <h2 class="modal-title">{{ editingMemberIndex !== null ? '멤버 수정' : '새 멤버 추가' }}</h2>
          <button class="close-btn" @click="closeAddMemberModal">&times;</button>
        </div>
        
        <form @submit.prevent="saveMember">
          <div class="form-group">
            <label class="form-label">이름 *</label>
            <input v-model="memberForm.name" class="form-input" type="text" required placeholder="멤버 이름을 입력하세요">
          </div>
          
          <div class="form-actions">
            <button type="button" class="btn btn-secondary" @click="closeAddMemberModal">취소</button>
            <button type="submit" class="btn btn-primary">{{ editingMemberIndex !== null ? '수정' : '추가' }}</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      currentView: 'board', // 'board', 'members', 'guide'
      columns: [
        { 
          id: 'backlog', 
          title: '백로그', 
          icon: 'fas fa-inbox', 
          tasks: [],
          description: '아이디어와 요구사항을 수집하는 단계',
          usage: '새로운 기능 아이디어, 버그 리포트, 개선 사항 등을 기록할 때',
          tips: '우선순위를 정하기 전 모든 아이디어를 자유롭게 추가하세요.'
        },
        { 
          id: 'todo', 
          title: '할 일', 
          icon: 'fas fa-list', 
          tasks: [],
          description: '작업 준비가 완료되어 시작을 기다리는 태스크',
          usage: '백로그에서 선별된 태스크들을 구체적으로 정의하고 우선순위를 매길 때',
          tips: '담당자와 마감일을 명확히 설정하여 작업 시작 시점을 명확히 하세요.'
        },
        { 
          id: 'inprogress', 
          title: '진행 중', 
          icon: 'fas fa-cog fa-spin', 
          tasks: [],
          description: '현재 활발히 작업 중인 태스크',
          usage: '팀원이 실제로 작업을 시작했을 때',
          tips: '동시에 너무 많은 태스크를 진행하지 않도록 WIP 제한을 두세요.'
        },
        { 
          id: 'review', 
          title: '리뷰', 
          icon: 'fas fa-eye', 
          tasks: [],
          description: '작업이 완료되어 검토와 승인을 기다리는 태스크',
          usage: '코드 리뷰, 품질 검사, 테스트가 필요한 작업이 완료되었을 때',
          tips: '명확한 리뷰 기준을 설정하여 빠른 피드백이 가능하도록 하세요.'
        },
        { 
          id: 'done', 
          title: '완료', 
          icon: 'fas fa-check-circle', 
          tasks: [],
          description: '모든 작업과 검토가 완료된 태스크',
          usage: '리뷰를 통과하고 실제 배포나 적용이 완료되었을 때',
          tips: '정기적으로 완료된 태스크들을 정리하여 성과를 확인하세요.'
        }
      ],
      showTaskModal: false,
      showAddMemberModal: false,
      editingTask: null,
      editingMemberIndex: null,
      currentColumnId: null,
      taskForm: {
        title: '',
        description: '',
        assignee: '',
        priority: 'medium',
        dueDate: ''
      },
      memberForm: {
        name: ''
      },
      teamMembers: ['김철수', '이영희', '박민수', '최지영', '정호석'],
      draggingTask: null,
      dragOverColumn: null,
      nextTaskId: 1,
      guideDetails: [
        {
          id: 'backlog',
          title: '백로그 (Backlog)',
          when: '아직 착수하지 않은 모든 아이디어, 요청사항, 예정된 작업을 저장할 때',
          role: '프로젝트의 작업 저장소 역할 — 차후 할일 컬럼으로 옮길 후보군을 보관합니다.',
          tips: [
            '한 번에 처리하기 힘든 요청도 일단 백로그에 기록해 누락을 방지하세요.',
            '정기적으로 백로그를 검토해 오래된 항목은 정리하거나 우선순위를 재설정하세요.'
          ]
        },
        {
          id: 'todo',
          title: '할일 (To Do)',
          when: '착수 우선순위가 정해져, 곧 작업을 시작할 준비가 되었을 때',
          role: '이번 주 또는 스프린트 내에 처리할 확정 작업 목록을 보관합니다. 진행 전 상태입니다.',
          tips: [
            '담당자와 마감일을 반드시 설정해 책임소재와 일정 관리를 명확히 하세요.',
            '항목이 너무 많아지면 집중이 분산되므로 우선순위가 높은 것만 유지하세요.'
          ]
        },
        {
          id: 'inprogress',
          title: '진행중 (In Progress)',
          when: '작업자가 해당 업무에 착수했을 때',
          role: '현재 진행 중인 업무의 현황판 역할을 하며, 팀원 별 작업 부하를 파악할 수 있습니다.',
          tips: [
            '한 사람이 동시에 너무 많은 태스크를 진행하지 않도록 WIP(작업중인 항목) 제한을 두세요.',
            '진행이 지연되면 원인을 빠르게 파악해 병목을 해소하세요.'
          ]
        },
        {
          id: 'review',
          title: '리뷰 (Review)',
          when: '작업이 완료되었지만 검토·테스트·코드 리뷰가 필요한 경우',
          role: '품질 보증을 위한 중간 확인 단계 — 테스트, 검토, 승인 절차를 진행합니다.',
          tips: [
            '리뷰어를 미리 지정해 대기 시간을 줄이세요.',
            '리뷰 코멘트는 명확히 작성하고, 수정 후 바로 완료 컬럼으로 이동시키세요.'
          ]
        },
        {
          id: 'done',
          title: '완료 (Done)',
          when: '모든 검토와 테스트를 마치고 최종 승인된 순간',
          role: '성과 저장소 — 팀의 완료된 작업을 시각적으로 확인하고 기록합니다.',
          tips: [
            '주기적으로 완료된 항목을 검토해 회고에 활용하세요.',
            '완료 항목은 리포트나 KPI 산출에 활용할 수 있습니다.'
          ]
        }
      ],
    }
  },
  
  computed: {
    totalTasks() {
      return this.columns.reduce((total, column) => total + column.tasks.length, 0);
    },
    
    inProgressTasks() {
      const inProgressColumn = this.columns.find(col => col.id === 'inprogress');
      return inProgressColumn ? inProgressColumn.tasks.length : 0;
    },
    
    completedTasks() {
      const doneColumn = this.columns.find(col => col.id === 'done');
      return doneColumn ? doneColumn.tasks.length : 0;
    },
    
    progressPercentage() {
      return this.totalTasks > 0 ? Math.round((this.completedTasks / this.totalTasks) * 100) : 0;
    }
  },
  
  mounted() {
    this.loadSampleData();
    this.loadFromStorage();
  },
  
  methods: {
    loadSampleData() {
      const sampleTasks = [
        {
          id: this.nextTaskId++,
          title: '프로젝트 기획서 작성',
          description: '새로운 웹 애플리케이션 프로젝트의 기획서를 작성합니다.',
          assignee: '김철수',
          priority: 'high',
          dueDate: '2024-01-15',
          columnId: 'todo'
        },
        {
          id: this.nextTaskId++,
          title: 'UI/UX 디자인',
          description: '사용자 인터페이스와 사용자 경험 디자인을 완성합니다.',
          assignee: '이영희',
          priority: 'medium',
          dueDate: '2024-01-20',
          columnId: 'inprogress'
        },
        {
          id: this.nextTaskId++,
          title: '백엔드 API 개발',
          description: 'RESTful API 서버를 구축합니다.',
          assignee: '박민수',
          priority: 'high',
          dueDate: '2024-01-25',
          columnId: 'inprogress'
        },
        {
          id: this.nextTaskId++,
          title: '데이터베이스 설계',
          description: '효율적인 데이터베이스 스키마를 설계합니다.',
          assignee: '최지영',
          priority: 'medium',
          dueDate: '2024-01-18',
          columnId: 'review'
        },
        {
          id: this.nextTaskId++,
          title: '프로토타입 제작',
          description: '초기 프로토타입을 제작하여 검증합니다.',
          assignee: '정호석',
          priority: 'low',
          dueDate: '2024-01-10',
          columnId: 'done'
        }
      ];
      
      sampleTasks.forEach(task => {
        const column = this.columns.find(col => col.id === task.columnId);
        if (column) {
          column.tasks.push(task);
        }
      });
    },
    
    // Task 관련 메서드들
    openTaskModal(columnId) {
      this.currentColumnId = columnId;
      this.showTaskModal = true;
      this.resetTaskForm();
    },
    
    closeTaskModal() {
      this.showTaskModal = false;
      this.editingTask = null;
      this.currentColumnId = null;
      this.resetTaskForm();
    },
    
    resetTaskForm() {
      this.taskForm = {
        title: '',
        description: '',
        assignee: '',
        priority: 'medium',
        dueDate: ''
      };
    },
    
    saveTask() {
      if (this.editingTask) {
        // 수정 모드
        Object.assign(this.editingTask, this.taskForm);
      } else {
        // 새 태스크 추가
        const newTask = {
          id: this.nextTaskId++,
          ...this.taskForm,
          createdAt: new Date().toISOString()
        };
        
        const column = this.columns.find(col => col.id === this.currentColumnId);
        if (column) {
          column.tasks.push(newTask);
        }
      }
      
      this.saveToStorage();
      this.closeTaskModal();
    },
    
    editTask(task) {
      this.editingTask = task;
      this.taskForm = { ...task };
      this.showTaskModal = true;
    },
    
    deleteTask(taskId) {
      if (confirm('이 태스크를 삭제하시겠습니까?')) {
        this.columns.forEach(column => {
          column.tasks = column.tasks.filter(task => task.id !== taskId);
        });
        this.saveToStorage();
      }
    },
    
    // Member 관련 메서드들
    closeAddMemberModal() {
      this.showAddMemberModal = false;
      this.editingMemberIndex = null;
      this.memberForm.name = '';
    },
    
    saveMember() {
      if (!this.memberForm.name.trim()) return;
      
      if (this.editingMemberIndex !== null) {
        // 수정 모드
        const oldName = this.teamMembers[this.editingMemberIndex];
        this.teamMembers[this.editingMemberIndex] = this.memberForm.name.trim();        
        // 기존 태스크의 담당자 이름도 업데이트
        this.updateTaskAssignee(oldName, this.memberForm.name.trim());
      } else {
        // 새 멤버 추가
        if (!this.teamMembers.includes(this.memberForm.name.trim())) {
          this.teamMembers.push(this.memberForm.name.trim());
        }
      }
      
      this.saveToStorage();
      this.closeAddMemberModal();
    },
    
    editMember(memberName, index) {
      this.editingMemberIndex = index;
      this.memberForm.name = memberName;
      this.showAddMemberModal = true;
    },
    
    deleteMember(index) {
      const memberName = this.teamMembers[index];
      const assignedTasks = this.getAssignedTaskCount(memberName);
      
      if (assignedTasks > 0) {
        if (!confirm(`${memberName}님에게 할당된 ${assignedTasks}개의 태스크가 있습니다. 정말 삭제하시겠습니까?`)) {
          return;
        }
      }
      
      this.teamMembers.splice(index, 1);
      
      // 해당 멤버가 할당된 태스크들의 담당자 정보 제거
      this.columns.forEach(column => {
        column.tasks.forEach(task => {
          if (task.assignee === memberName) {
            task.assignee = '';
          }
        });
      });
      
      this.saveToStorage();
    },
    
    updateTaskAssignee(oldName, newName) {
      this.columns.forEach(column => {
        column.tasks.forEach(task => {
          if (task.assignee === oldName) {
            task.assignee = newName;
          }
        });
      });
    },
    
    getAssignedTaskCount(memberName) {
      let count = 0;
      this.columns.forEach(column => {
        count += column.tasks.filter(task => task.assignee === memberName).length;
      });
      return count;
    },
    
    // Drag & Drop 관련 메서드들
    handleDragStart(event, task) {
      this.draggingTask = task.id;
      event.dataTransfer.effectAllowed = 'move';
      event.dataTransfer.setData('application/json', JSON.stringify(task));
    },
    
    handleDragEnd() {
      this.draggingTask = null;
      this.dragOverColumn = null;
    },
    
    handleDragOver(columnId) {
      this.dragOverColumn = columnId;
    },
    
    handleDragLeave() {
      this.dragOverColumn = null;
    },
    
    handleDrop(event, targetColumnId) {
      event.preventDefault();
      
      try {
        const taskData = JSON.parse(event.dataTransfer.getData('application/json'));        
        // 원래 컬럼에서 태스크 제거
        this.columns.forEach(column => {
          column.tasks = column.tasks.filter(task => task.id !== taskData.id);
        });        
        // 새 컬럼에 태스크 추가
        const targetColumn = this.columns.find(col => col.id === targetColumnId);
        if (targetColumn) {
          targetColumn.tasks.push(taskData);
        }        
        this.saveToStorage();
      } catch (error) {
        console.error('드래그 앤 드롭 오류:', error);
      }
      
      this.dragOverColumn = null;
      this.draggingTask = null;
    },
    
    // 유틸리티 메서드들
    getPriorityText(priority) {
      const priorityMap = {
        low: '낮음',
        medium: '보통',
        high: '높음'
      };
      return priorityMap[priority] || '보통';
    },
    
    formatDate(dateString) {
      if (!dateString) return '';
      const date = new Date(dateString);
      return date.toLocaleDateString('ko-KR');
    },
    
    saveToStorage() {
      try {
        const data = {
          columns: this.columns,
          nextTaskId: this.nextTaskId,
          teamMembers: this.teamMembers
        };
        localStorage.setItem('kanbanData', JSON.stringify(data));
      } catch (error) {
        console.error('저장 오류:', error);
      }
    },
    
    loadFromStorage() {
      try {
        const savedData = localStorage.getItem('kanbanData');
        if (savedData) {
          const data = JSON.parse(savedData);
          if (data.columns) this.columns = data.columns;
          if (data.nextTaskId) this.nextTaskId = data.nextTaskId;
          if (data.teamMembers) this.teamMembers = data.teamMembers;
        }
      } catch (error) {
        console.error('로드 오류:', error);
      }
    }
  }
}
</script>

<style scoped>
* {  margin: 0;  padding: 0;  box-sizing: border-box;  }
.app {
  min-height: 100vh;  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;  color: #333;
}

/* 네비게이션 헤더 */
.nav-header {
  background: rgba(255, 255, 255, 0.95);  backdrop-filter: blur(10px);  border-bottom: 1px solid rgba(255, 255, 255, 0.18);  padding: 0 20px;
}
.nav-tabs {  
  display: flex;  gap: 0;
}
.nav-tab {
  background: none;  border: none;  padding: 15px 25px;  font-size: 1rem;  font-weight: 500;  color: #7f8c8d;  cursor: pointer;
  transition: all 0.3s ease;  border-bottom: 3px solid transparent;  display: flex;  align-items: center;  gap: 8px;
}
.nav-tab:hover {
  color: #2c3e50;  background: rgba(52, 152, 219, 0.1);
}
.nav-tab.active {
  color: #3498db;  border-bottom-color: #3498db;  background: rgba(52, 152, 219, 0.1);
}
/* 공통 뷰 스타일 */
.board-view, .members-view, .guide-view {
  padding: 20px;
}
.header {
  background: rgba(255, 255, 255, 0.95);  padding: 20px 30px;  border-radius: 15px;  margin-bottom: 30px;  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);  border: 1px solid rgba(255, 255, 255, 0.18);
}
.header h1 {
  color: #2c3e50;  font-size: 2rem;  font-weight: 700;  margin-bottom: 10px;
}
.header p {
  color: #7f8c8d;  font-size: 1.1rem;
}
.stats {
  display: flex;  gap: 15px;  margin-top: 15px;  flex-wrap: wrap;
}
.stat-item {
  background: rgba(255, 255, 255, 0.2);  padding: 10px 15px;  border-radius: 10px;  color: #036;  font-size: 0.9rem;
}
.progress-bar {
  width: 100%;  height: 6px;  background: rgba(255, 255, 255, 0.2);  border-radius: 3px;  margin-top: 10px;  overflow: hidden;
}
.progress-fill {
  height: 100%;  background: #27ae60;  transition: width 0.3s ease;
}
/* 칸반 보드 스타일 */
.board {
  display: flex;  gap: 20px;  overflow-x: auto;  padding-bottom: 20px;  min-height: calc(100vh - 300px);
}
.column {
  min-width: 300px;  background: rgba(255, 255, 255, 0.95);  border-radius: 15px;  padding: 20px;  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);  border: 1px solid rgba(255, 255, 255, 0.18);  height: fit-content;
}
.column-header {
  display: flex;  justify-content: space-between;  align-items: center;  margin-bottom: 15px;  padding-bottom: 15px;  border-bottom: 2px solid #ecf0f1;
}
.column-title {
  font-size: 1.3rem;  font-weight: 600;  color: #2c3e50;  display: flex;  align-items: center;  gap: 10px;
}
.column-description {
  color: #7f8c8d;  font-size: 0.9rem;  margin-bottom: 20px;  font-style: italic;  line-height: 1.4;
}
.task-count {
  background: #3498db;  color: #fff;  border-radius: 12px;  padding: 4px 8px;  font-size: 0.8rem;  font-weight: 600;
}
.add-task-btn {
  background: #27ae60;  color: #fff;  border: none;  border-radius: 8px;  padding: 8px 12px;  cursor: pointer;  font-size: 0.9rem;
  transition: all 0.3s ease;  display: flex;  align-items: center;  gap: 5px;
}
.add-task-btn:hover {
  background: #219a52;  transform: translateY(-2px);
}
.tasks-container {
  min-height: 100px;  transition: all 0.3s ease;
}
.drag-over {
  background: rgba(52, 152, 219, 0.1);  border: 2px dashed #3498db;  border-radius: 10px;
}
.empty-state {
  text-align: center;  color: #95a5a6;  padding: 40px 20px;  font-style: italic;
}
.task-card {
  background: #fff;  border-radius: 12px;  padding: 16px;  margin-bottom: 12px;  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);  cursor: grab;
  transition: all 0.3s ease;  border: 1px solid #ecf0f1;
}
.task-card:hover {
  transform: translateY(-2px);  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.15);
}
.task-card.dragging {
  opacity: 0.5;  transform: rotate(5deg);
}
.task-title {
  font-weight: 600;  color: #2c3e50;  margin-bottom: 8px;  font-size: 1rem;
}
.task-description {
  color: #7f8c8d;  font-size: 0.9rem;  line-height: 1.4;  margin-bottom: 12px;
}
.task-meta {
  display: flex;  justify-content: space-between;  align-items: center;  flex-wrap: wrap;  gap: 8px;
}
.task-assignee {
  background: #e8f6ff;  color: #3498db;  padding: 4px 8px;  border-radius: 6px;  font-size: 0.8rem;  font-weight: 500;
}
.task-priority {
  padding: 4px 8px;  border-radius: 6px;  font-size: 0.8rem;  font-weight: 600;
}
.priority-high {
  background: #fee;  color: #e74c3c;
}
.priority-medium {
  background: #fff4e6;  color: #f39c12;
}
.priority-low {
  background: #f0fff4;  color: #27ae60;
}
.task-date {
  color: #95a5a6;  font-size: 0.8rem;  margin-top: 8px;
}
.task-actions {
  display: flex;  gap: 8px;  margin-top: 12px;
}
.task-action-btn {
  background: #f8f9fa;  border: 1px solid #e9ecef;  color: #6c757d;  cursor: pointer;  padding: 6px 8px;  border-radius: 6px;  transition: all 0.3s ease;
  font-size: 0.85rem;  min-width: 32px;  height: 32px;  display: flex;  align-items: center;  justify-content: center;
}
.task-action-btn:hover {
  background: #e9ecef;  color: #495057;  transform: translateY(-1px);
}
.task-action-btn.delete-btn:hover {
  background: #f8d7da;  color: #721c24;  border-color: #f5c6cb;
}
/* 담당자 관리 스타일 */
.members-container {
  background: rgba(255, 255, 255, 0.95);  border-radius: 15px;  padding: 30px;  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);  border: 1px solid rgba(255, 255, 255, 0.18);
}
.members-header {
  display: flex;  justify-content: space-between;  align-items: center;  margin-bottom: 30px;  padding-bottom: 20px;  border-bottom: 2px solid #ecf0f1;
}
.members-header h2 {
  color: #2c3e50;  font-size: 1.5rem;  font-weight: 600;
}
.add-member-btn {
  background: #3498db;  color: #fff;  border: none;  border-radius: 10px;  padding: 12px 20px;  cursor: pointer;  font-size: 1rem;  font-weight: 500;
  transition: all 0.3s ease;  display: flex;  align-items: center;  gap: 8px;
}
.add-member-btn:hover {
  background: #2980b9;  transform: translateY(-2px);
}
.members-grid {
  display: grid;  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));  gap: 20px;
}
.member-card {
  background: #fff;  border-radius: 12px;  padding: 20px;  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);  transition: all 0.3s ease;  border: 1px solid #ecf0f1;
  display: flex;  align-items: center;  gap: 15px;
}
.member-card:hover {
  transform: translateY(-2px);  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.15);
}
.member-avatar {
  font-size: 2.5rem;  color: #3498db;
}
.member-info {
  flex: 1;
}
.member-name {
  font-size: 1.1rem;  font-weight: 600;  color: #2c3e50;  margin-bottom: 5px;
}
.member-stats {
  color: #7f8c8d;  font-size: 0.9rem;
}
.member-actions {
  display: flex;  gap: 8px;
}
.member-action-btn {
  background: none;  border: none;  padding: 8px;  border-radius: 6px;  cursor: pointer;  font-size: 0.9rem;  transition: all 0.3s ease;
}
.edit-btn {
  color: #3498db;
}
.edit-btn:hover {
  background: #e8f6ff;
}
.delete-btn {
  color: #e74c3c;
}
.delete-btn:hover {
  background: #fee;
}
.empty-members {
  text-align: center;  padding: 60px 20px;  color: #95a5a6;
}
.empty-members i {
  font-size: 4rem;  margin-bottom: 20px;  color: #bdc3c7;
}
.empty-members h3 {
  margin-bottom: 10px;  color: #7f8c8d;
}
/* 가이드 뷰 스타일 */
.guide-container {
  display: grid;  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));  gap: 20px;
}
.guide-card {
  background: rgba(255, 255, 255, 0.95);  border-radius: 15px;  padding: 25px;  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.18);  transition: all 0.3s ease;
}
.guide-card:hover {
  transform: translateY(-5px);  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.15);
}
.guide-header {
  display: flex;  align-items: center;  gap: 12px;  margin-bottom: 20px;  padding-bottom: 15px;  border-bottom: 2px solid #ecf0f1;
}
.guide-header i {
  font-size: 1.5rem;  color: #3498db;
}
.guide-header h3 {
  color: #2c3e50;  font-size: 1.3rem;  font-weight: 600;
}
.guide-content {
  line-height: 1.6;
}
.guide-description {
  color: #34495e;  margin-bottom: 20px;  font-size: 1rem;
}
.guide-usage, .guide-tips {
  margin-bottom: 15px;
}
.guide-usage h4, .guide-tips h4 {
  color: #2c3e50;  font-size: 0.95rem;  font-weight: 600;  margin-bottom: 8px;
}
.guide-usage p, .guide-tips p {
  color: #7f8c8d;  font-size: 0.9rem;
}
/* 모달 스타일 */
.modal {
  position: fixed;  top: 0;  left: 0;  width: 100%;  height: 100%;  background: rgba(0, 0, 0, 0.5);  display: flex;  justify-content: center;
  align-items: center;  z-index: 1000;
}
.modal-content {
  background: #fff;  border-radius: 15px;  padding: 30px;  width: 90%;  max-width: 500px;  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  max-height: 90vh;  overflow-y: auto;
}
.modal-header {
  display: flex;  justify-content: space-between;  align-items: center;  margin-bottom: 20px;
}
.modal-title {
  font-size: 1.5rem;  font-weight: 600;  color: #2c3e50;
}
.close-btn {
  background: none;  border: none;  font-size: 1.5rem;  color: #95a5a6;  cursor: pointer;  padding: 5px;
}
.close-btn:hover {
  color: #2c3e50;
}
.form-group {
  margin-bottom: 20px;
}
.form-label {
  display: block;  font-weight: 600;  color: #2c3e50;  margin-bottom: 8px;
}
.form-input, .form-textarea, .form-select {
  width: 100%;  padding: 12px;  border: 2px solid #ecf0f1;  border-radius: 8px;  font-size: 1rem;  transition: border-color 0.3s ease;  font-family: inherit;
}
.form-input:focus, .form-textarea:focus, .form-select:focus {
  outline: none;  border-color: #3498db;
}
.form-textarea {
  resize: vertical;  min-height: 80px;
}
.form-actions {
  display: flex;  gap: 12px;  justify-content: flex-end;
}
.btn {
  padding: 12px 24px;  border: none;  border-radius: 8px;  cursor: pointer;  font-size: 1rem;  font-weight: 500;  transition: all 0.3s ease;
}
.btn-primary {
  background: #3498db;  color: white;
}
.btn-primary:hover {
  background: #2980b9;
}
.btn-secondary {
  background: #95a5a6;  color: #fff;
}
.btn-secondary:hover {
  background: #7f8c8d;
}
/* 반응형 디자인 */
@media (max-width: 768px) {
  .nav-tabs {
    overflow-x: auto;
  }  
  .nav-tab {
    white-space: nowrap;    min-width: fit-content;
  }  
  .board {
    flex-direction: column;
  }  
  .column {
    min-width: auto;    width: 100%;
  }  
  .header {
    padding: 15px 20px;
  }  
  .header h1 {
    font-size: 1.5rem;
  }
  .stats {
    flex-direction: column;    gap: 10px;
  }  
  .members-grid {
    grid-template-columns: 1fr;
  }  
  .members-header {
    flex-direction: column;    gap: 15px;    align-items: stretch;
  }  
  .guide-container {
    grid-template-columns: 1fr;
  }  
  .modal-content {
    margin: 20px;    padding: 20px;
  }
}
</style>