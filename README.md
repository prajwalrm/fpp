describe('ConfirmationsSupervisoryDashboard', () => {
  let component: ConfirmationsSupervisoryDashboard;
  let mockSupervisionService;
  let mockSpinnerService;
  let mockTableService;

  beforeEach(() => {
    // Mock the required services
    mockSupervisionService = jasmine.createSpyObj('ConfirmationsSupervisionService', [
      'getEventAgeAggregationTable'
    ]);
    mockSpinnerService = jasmine.createSpyObj('QztSpinnerService', ['spin', 'stop']);
    mockTableService = jasmine.createSpyObj('QztTableService', ['createDataSource']);

    component = new ConfirmationsSupervisoryDashboard(
      jasmine.createSpyObj('ChangeDetectorRef', ['markForCheck']),
      { Services: { QzDocsSupervision: 'mockService' } },
      mockSpinnerService,
      jasmine.createSpyObj('FileService', []),
      jasmine.createSpyObj('TradeService', []),
      jasmine.createSpyObj('ModalService', []),
      jasmine.createSpyObj('TaskService', []),
      mockTableService,
      mockSupervisionService
    );
  });

  it('should create the component', () => {
    expect(component).toBeTruthy();
  });

  // Tests for ngOnInit
  describe('ngOnInit', () => {
    it('should initialize data sources and call aggregation methods', () => {
      spyOn(component, 'getLeftAggregation');
      spyOn(component, 'getRightAggregation');
      spyOn(component, 'refreshCallback');

      component.ngOnInit();

      expect(component.btsource).toBeTruthy();
      expect(component.cfsource).toBeTruthy();
      expect(component.getLeftAggregation).toHaveBeenCalled();
      expect(component.getRightAggregation).toHaveBeenCalled();
      expect(component.refreshCallback).toHaveBeenCalled();
    });
  });

  // Tests for getLeftAggregation
  describe('getLeftAggregation', () => {
    it('should call getEventAgeAggregationTable with correct parameters', () => {
      component.leftTableState = { pagination: { start: 0, totalItemCount: 0 }, sort: {} };
      component.viewMap = { Team: 'Team' };
      component.leftView1 = 'Team';
      component.leftView2 = 'Assignee';

      mockSupervisionService.getEventAgeAggregationTable.and.returnValue(of({
        TotalPages: 2,
        totalItemCount: 100,
        Table: [],
        filterCount: 5
      }));

      component.getLeftAggregation();

      expect(mockSpinnerService.spin).toHaveBeenCalled();
      expect(mockSupervisionService.getEventAgeAggregationTable).toHaveBeenCalledWith(
        component.leftTableState,
        component.perspective,
        'Team',
        'Assignee',
        null,
        component.getFilterOptions(),
        component.leftTableState.sort
      );
    });

    it('should update left source with response data', () => {
      mockSupervisionService.getEventAgeAggregationTable.and.returnValue(of({
        TotalPages: 2,
        totalItemCount: 100,
        Table: [{ name: 'TestRow' }],
        filterCount: 5
      }));

      component.getLeftAggregation();

      expect(component.leftSource).toEqual([{ name: 'TestRow' }]);
    });
  });

  // Tests for getRightAggregation
  describe('getRightAggregation', () => {
    it('should call getEventAgeAggregationTable for right aggregation', () => {
      component.rightTableState = { pagination: { start: 0, totalItemCount: 0 }, sort: {} };
      component.viewMap = { Team: 'Team' };
      component.rightView1 = 'Priority';
      component.rightView2 = 'Assignee';

      mockSupervisionService.getEventAgeAggregationTable.and.returnValue(of({
        TotalPages: 3,
        totalItemCount: 150,
        Table: [],
        filterCount: 10
      }));

      component.getRightAggregation();

      expect(mockSpinnerService.spin).toHaveBeenCalled();
      expect(mockSupervisionService.getEventAgeAggregationTable).toHaveBeenCalledWith(
        component.rightTableState,
        component.perspective,
        'Priority',
        'Assignee',
        null,
        component.getFilterOptions(),
        component.rightTableState.sort
      );
    });
  });

  // Tests for selectLeftView1
  describe('selectLeftView1', () => {
    it('should update leftView1 and trigger getLeftAggregation', () => {
      spyOn(component, 'getLeftAggregation');

      component.selectLeftView1('Priority');

      expect(component.leftView1).toBe('Priority');
      expect(component.getLeftAggregation).toHaveBeenCalled();
    });
  });

  // Tests for selectRightView1
  describe('selectRightView1', () => {
    it('should update rightView1 and trigger getRightAggregation', () => {
      spyOn(component, 'getRightAggregation');

      component.selectRightView1('Team');

      expect(component.rightView1).toBe('Team');
      expect(component.getRightAggregation).toHaveBeenCalled();
    });
  });

  // Tests for clickFilterOption
  describe('clickFilterOption', () => {
    it('should toggle filter button states and refresh aggregations', () => {
      spyOn(component, 'refreshAggregations');
      component.buttonStates = { All: true, ZeroToSevenDays: false };

      component.clickFilterOption('ZeroToSevenDays');

      expect(component.buttonStates['All']).toBe(false);
      expect(component.buttonStates['ZeroToSevenDays']).toBe(true);
      expect(component.refreshAggregations).toHaveBeenCalled();
    });
  });

  // Tests for getFilterOptions
  describe('getFilterOptions', () => {
    it('should return selected filter options', () => {
      component.buttonStates = { All: false, ZeroToSevenDays: true, EightToFourteenDays: true };
      const filters = component.getFilterOptions();

      expect(filters).toEqual(['ZeroToSevenDays', 'EightToFourteenDays']);
    });
  });
});

describe('ConfirmationsSupervisoryDashboard Additional Methods', () => {
  let component: ConfirmationsSupervisoryDashboard;
  let mockTableService, mockSpinnerService, mockTaskService, mockModalService;

  beforeEach(() => {
    // Mock dependencies
    mockTableService = jasmine.createSpyObj('QztTableService', ['createDataSource']);
    mockSpinnerService = jasmine.createSpyObj('QztSpinnerService', ['spin', 'stop']);
    mockTaskService = jasmine.createSpyObj('QztTaskService', ['loadTask']);
    mockModalService = jasmine.createSpyObj('QztModalService', ['openX1']);

    component = new ConfirmationsSupervisoryDashboard(
      jasmine.createSpyObj('ChangeDetectorRef', ['markForCheck']),
      { Services: { QzDocsSupervision: 'mockService' } },
      mockSpinnerService,
      jasmine.createSpyObj('FileService', []),
      jasmine.createSpyObj('TradeService', []),
      mockModalService,
      mockTaskService,
      mockTableService,
      jasmine.createSpyObj('ConfirmationsSupervisionService', [])
    );
  });

  it('should create the component', () => {
    expect(component).toBeTruthy();
  });

  // Tests for refreshSubTaskTable
  describe('refreshSubTaskTable', () => {
    it('should configure the data source and refresh it', () => {
      const mockCfsource = { refresh: jasmine.createSpy('refresh') };
      const filterOptions = ['ZeroToSevenDays'];
      component.getFilterOptions = jasmine.createSpy('getFilterOptions').and.returnValue(filterOptions);
      mockTableService.createDataSource.and.returnValue(mockCfsource);

      component.refreshSubTaskTable();

      expect(mockTableService.createDataSource).toHaveBeenCalledWith('TaskTableConfirmations', {
        service_name: component.env.Services.QzDocsSupervision,
        url: 'team/test2',
        params: {
          Tasktype: 'Subtask',
          perspective: component.perspective,
          bucket: component.bucket,
          options: JSON.stringify(filterOptions),
          rightlinkInfo: component.rightlinkInfo,
          leftlinkInfo: component.leftlinkInfo,
          aggrFilter: component.getAggrFilter()
        },
        perPage: component.detailPaginationLevel
      });
      expect(mockCfsource.refresh).toHaveBeenCalled();
    });
  });

  // Tests for selectLeftView1
  describe('selectLeftView1', () => {
    it('should update leftView1 and call selectLeftView', () => {
      spyOn(component, 'selectLeftView');
      component.selectLeftView1('Priority');
      expect(component.leftView1).toBe('Priority');
      expect(component.selectLeftView).toHaveBeenCalled();
    });
  });

  // Tests for selectLeftView2
  describe('selectLeftView2', () => {
    it('should update leftView2 and call selectLeftView', () => {
      spyOn(component, 'selectLeftView');
      component.selectLeftView2('Team');
      expect(component.leftView2).toBe('Team');
      expect(component.selectLeftView).toHaveBeenCalled();
    });
  });

  // Tests for selectLeftView
  describe('selectLeftView', () => {
    it('should reset pagination and call getLeftAggregation', () => {
      spyOn(component, 'getLeftAggregation');
      component.leftTableState = { pagination: { start: 5 } };
      component.selectLeftView();
      expect(component.leftTableState.pagination.start).toBe(0);
      expect(component.getLeftAggregation).toHaveBeenCalled();
    });
  });

  // Tests for selectPerspective
  describe('selectPerspective', () => {
    it('should update perspective and refresh state', () => {
      spyOn(component, 'refreshState');
      component.selectPerspective('newPerspective');
      expect(component.perspective).toBe('newPerspective');
      expect(component.refreshState).toHaveBeenCalled();
    });
  });

  // Tests for selectTaskType
  describe('selectTaskType', () => {
    it('should update taskType and refresh state', () => {
      spyOn(component, 'refreshState');
      component.selectTaskType('newTaskType');
      expect(component.taskType).toBe('newTaskType');
      expect(component.refreshState).toHaveBeenCalled();
    });
  });

  // Tests for selectRightView1
  describe('selectRightView1', () => {
    it('should update rightView1 and call selectRightView', () => {
      spyOn(component, 'selectRightView');
      component.selectRightView1('Priority');
      expect(component.rightView1).toBe('Priority');
      expect(component.selectRightView).toHaveBeenCalled();
    });
  });

  // Tests for selectRightView2
  describe('selectRightView2', () => {
    it('should update rightView2 and call selectRightView', () => {
      spyOn(component, 'selectRightView');
      component.selectRightView2('Team');
      expect(component.rightView2).toBe('Team');
      expect(component.selectRightView).toHaveBeenCalled();
    });
  });

  // Tests for displayTaskScreen
  describe('displayTaskScreen', () => {
    it('should spin the spinner, load the task, and open a modal on success', () => {
      const mockTask = { id: 1, name: 'TestTask' };
      mockTaskService.loadTask.and.returnValue(of(mockTask));
      const event = { row: { data: { WorkflowProcessName: 'process', TaskId: '123' } } };

      component.displayTaskScreen(event);

      expect(mockSpinnerService.spin).toHaveBeenCalled();
      expect(mockTaskService.loadTask).toHaveBeenCalledWith('process', '123');
      expect(mockModalService.openX1).toHaveBeenCalledWith(QztTaskModal, { data: { task: mockTask } });
      expect(mockSpinnerService.stop).toHaveBeenCalled();
    });

    it('should stop the spinner on error', () => {
      mockTaskService.loadTask.and.returnValue(throwError('Error'));
      const event = { row: { data: { WorkflowProcessName: 'process', TaskId: '123' } } };

      component.displayTaskScreen(event);

      expect(mockSpinnerService.spin).toHaveBeenCalled();
      expect(mockSpinnerService.stop).toHaveBeenCalled();
    });
  });

  // Tests for tableForDownload
  describe('tableForDownload', () => {
    it('should return the correct table based on detailType', () => {
      component.detailType = 'CashFlow';
      component.cftable = 'MockCFTable';
      expect(component._tableForDownload()).toBe('MockCFTable');

      component.detailType = 'NetFlow';
      component.nftable = 'MockNFTable';
      expect(component._tableForDownload()).toBe('MockNFTable');

      component.detailType = 'BankTransaction';
      component.bttable = 'MockBTTable';
      expect(component._tableForDownload()).toBe('MockBTTable');
    });
  });
});
