---
title: Binding hierarchical grids
page_title: Binding hierarchical grids | UI for ASP.NET AJAX Documentation
description: Binding hierarchical grids
slug: grid/hierarchical-grid-types-and-load-modes/binding-hierarchical-grids
tags: binding,hierarchical,grids
published: True
position: 2
---

# Binding hierarchical grids



There are two general steps to implement a hierarchical grid.

## 1. Design the Grid Hierarchy

The first step when creating hierarchical grids is to specify the structure of the hierarchical tables. This step must be completed before binding the data.

This is related to populating the collection of __DetailTables__ of __RadGrid__.__MasterTableView__ and the respective columns. It is important that this step be completed __before__ RadGrid is bound to any data. For example, a grid can have a structure like this:

````XML
	  - MasterTableView
	    - DetailTable(0)
	        - DetailTable(0, 0)
````



__MasterTableView__ and __DetailTable__ are controls of type __GridTableView__. They are represented by an HTML table (runtime) object. This object has a __Column__, __AutoGeneratedColumn__ collections, properties for setting styles, etc.

The whole structure of __RadGrid__ with its detail tables is saved in the __ViewState__. If you build the grid programmatically, you can safely define the structure in __Page_Load__ event handler if there was no PostBack.



````C#
	    private void DefineGridStructure()
	    {
	        this.RadGrid1 = new RadGrid();
	
	        this.RadGrid1.NeedDataSource += new GridNeedDataSourceEventHandler(this.RadGrid1_NeedDataSource);
	        this.RadGrid1.DetailTableDataBind += new GridDetailTableDataBindEventHandler(this.RadGrid1_DetailTableDataBind);
	
	        this.RadGrid1.CssClass = "RadGrid";
	
	        this.RadGrid1.Width = Unit.Percentage(100);
	        this.RadGrid1.PageSize = 5;
	        this.RadGrid1.AllowPaging = true;
	        this.RadGrid1.AutoGenerateColumns = false;
	
	        this.RadGrid1.MasterTableView.DataMember = "Customers";
	        this.RadGrid1.MasterTableView.PageSize = 15;
	
	        GridBoundColumn boundColumn;
	        boundColumn = new GridBoundColumn();
	        boundColumn.DataField = "CustomerID";
	        boundColumn.HeaderText = "CustomerID";
	        this.RadGrid1.MasterTableView.Columns.Add(boundColumn);
	
	        boundColumn = new GridBoundColumn();
	        boundColumn.DataField = "ContactName";
	        boundColumn.HeaderText = "Contact Name";
	        this.RadGrid1.MasterTableView.Columns.Add(boundColumn);
	
	        this.RadGrid1.MasterTableView.Columns.Add(new GridExpandColumn());
	
	        this.RadGrid1.MasterTableView.GroupByExpressions.Add(new GridGroupByExpression("Country Group By Country"));
	
	        GridTableView tableViewOrders = new GridTableView(RadGrid1);
	        tableViewOrders.DataMember = "Orders";
	        this.RadGrid1.MasterTableView.DetailTables.Add(tableViewOrders);
	
	        boundColumn = new GridBoundColumn();
	        boundColumn.DataField = "OrderID";
	        boundColumn.HeaderText = "OrderID";
	        tableViewOrders.Columns.Add(boundColumn);
	
	        boundColumn = new GridBoundColumn();
	        boundColumn.DataField = "OrderDate";
	        boundColumn.HeaderText = "Date Ordered";
	        tableViewOrders.Columns.Add(boundColumn);
	
	        GridTableView tableViewOrderDetails = new GridTableView(RadGrid1);
	        tableViewOrderDetails.DataMember = "OrderDetails";
	        tableViewOrders.DetailTables.Add(tableViewOrderDetails);
	
	        boundColumn = new GridBoundColumn();
	        boundColumn.DataField = "UnitPrice";
	        boundColumn.HeaderText = "Unit Price";
	        tableViewOrderDetails.Columns.Add(boundColumn);
	
	        boundColumn = new GridBoundColumn();
	        boundColumn.DataField = "Quantity";
	        boundColumn.HeaderText = "Quantity";
	        tableViewOrderDetails.Columns.Add(boundColumn);
	
	        this.PlaceHolder1.Controls.Add(RadGrid1);
	    }
````
````VB.NET
	    Private Sub DefineGridStructure()
	        Me.RadGrid1 = New RadGrid
	
	        AddHandler Me.RadGrid1.NeedDataSource, New GridNeedDataSourceEventHandler(AddressOf Me.RadGrid1_NeedDataSource)
	        AddHandler Me.RadGrid1.DetailTableDataBind, New GridDetailTableDataBindEventHandler(AddressOf Me.RadGrid1_DetailTableDataBind)
	
	        Me.RadGrid1.CssClass = "RadGrid"
	        Me.RadGrid1.Width = Unit.Percentage(100)
	        Me.RadGrid1.PageSize = 5
	        Me.RadGrid1.AllowPaging = True
	        Me.RadGrid1.AutoGenerateColumns = False
	        Me.RadGrid1.MasterTableView.DataMember = "Customers"
	        Me.RadGrid1.MasterTableView.PageSize = 15
	
	        Dim column1 As New GridBoundColumn
	        column1.DataField = "CustomerID"
	        column1.HeaderText = "CustomerID"
	        Me.RadGrid1.MasterTableView.Columns.Add(column1)
	
	        column1 = New GridBoundColumn
	        column1.DataField = "ContactName"
	        column1.HeaderText = "Contact Name"
	        Me.RadGrid1.MasterTableView.Columns.Add(column1)
	
	        Dim view1 As New GridTableView(Me.RadGrid1)
	        view1.DataMember = "Orders"
	        Me.RadGrid1.MasterTableView.DetailTables.Add(view1)
	
	        column1 = New GridBoundColumn
	        column1.DataField = "OrderID"
	        column1.HeaderText = "OrderID"
	        view1.Columns.Add(column1)
	
	        column1 = New GridBoundColumn
	        column1.DataField = "OrderDate"
	        column1.HeaderText = "Date Ordered"
	        view1.Columns.Add(column1)
	
	        Dim view2 As New GridTableView(Me.RadGrid1)
	        view2.DataMember = "OrderDetails"
	        view1.DetailTables.Add(view2)
	
	        column1 = New GridBoundColumn
	        column1.DataField = "UnitPrice"
	        column1.HeaderText = "Unit Price"
	        view2.Columns.Add(column1)
	
	        column1 = New GridBoundColumn
	        column1.DataField = "Quantity"
	        column1.HeaderText = "Quantity"
	        view2.Columns.Add(column1)
	
	        Me.PlaceHolder1.Controls.Add(Me.RadGrid1)
	
	    End Sub
````


[Creating Hierarchy Programmatically demo](http://demos.telerik.com/aspnet-ajax/grid/examples/programming/hierarchy/defaultcs.aspx)

## 2. Binding Detail Tables Runtime

At runtime, when binding the data, __RadGrid__ builds the structure of items (grid rows) with respect to the structure of the detail tables as it was defined in Step 1. __RadGrid__ will create a new control __GridTableView__ for each item in the master table. Then __RadGrid__ will assign it as a child of the master item. This control will be a copy of the corresponding table - this should be DetailTable(0). So, for example, if you have 3 items in the __MasterTableView__ in the second level of the hierarchy, the grid will look like this:

````XML
	  MasterTableView
	    + Item(0)
	        DetailTable0 (copy 0)
	    + Item(1)
	        DetailTable0 (copy 1)
	    + Item(2)
	        DetailTable0 (copy 2)
````



In this structure Item(0) is the parent item forDetailTable(0). You can get its instance using the __GridTableView.ParentItem__ property, where __GridTableView__ is *DetailTable0 (copy 0)*. In our example grid structure there is third level of the hierarchy and each item of the copies of *DetailTable(0)* should have a detail table that is copy of DetailTable(0, 0). For example let's say that the copy of DetailTable(0) with index 0 has 2 items:

````XML
	  DetailTable0 (copy 0)
	    + Item(0)
	        DetailTable(0, 0) (copy 0)
	    + Item(1)
	        DetailTable(0, 0) (copy 1)
````



Each copy of the detail tables should be bound in order to have its item populated. __RadGrid__ delegates the full control of how this should be done to the developer, i.e. you should only filter or select the records that correspond to each detail table. The only restriction is when this should happen. This is the __DetailTableDataBind__ event. __RadGrid__ fires this event for each __DetailTable__ that is about to be bound. Firing of this event also depends on the [[!] HierarchyLoadMode](hierarchy-load-modes.html) of the corresponding __GridTableView__ as stated when it is declared in Step 1.

* If the __HierarchyLoadMode__ is set to __ServerBind__, then the __DetailTableDataBind__ event will be fired immediately after the corresponding parent item is bound.

* If it is __ServerOnDemand__, the detail table will be bound as soon as its parent item is expanded.

You can determine which detail table to be databound using its __DataMember__ property, that should be properly assigned in Step 1.

When __RadGrid__ constructs each copy of the detail tables, it assigns its data-source of the grid itself. This is useful when the data source is a __DataSet__ containing all the detail table data.Using the __DetailTableDataBind()__method, you can obtain the instance of the __DataSet__ from the __DataSource__ property of the detail table that is an argument of the event handler (referred to as __e.DetailTable__). Prior to rebinding __MasterTableView__ or any detail tables, __RadGrid__ fires __NeedDataSource__ event, so the developer can assign a __DataSource__. Note that it will be fired only once for a group of detail tables and only if the __DataSource__ of __RadGrid__ is not already assigned.