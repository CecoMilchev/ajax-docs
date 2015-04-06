---
title: Optimizing ViewState usage
page_title: Optimizing ViewState usage | UI for ASP.NET AJAX Documentation
description: Optimizing ViewState usage
slug: grid/performance/optimizing-viewstate-usage
tags: optimizing,viewstate,usage
published: True
position: 3
---

# Optimizing ViewState usage



## 

Typically RadGrid stores in the __ViewState__ only items/controls collections. However, sometimes the page __ViewState__ can grow too big and might significantly increase the page download time.

You can control this behavior by setting the __EnableViewState__ property of RadGrid to false if you do not wish the data for the controls in the grid to to be persisted in the __ViewState__. This means that the control will need to be rebound on every request: either by firing the NeedDataSource event or by going through an ASP.NET 2.x/3.x data source control.

>note When you set the __EnableViewState__ property of __RadGrid__ to __False__ , the only supported way for binding is touse Advanced DataBinding (using[NeedDataSource event ](http://www.telerik.com/help/aspnet-ajax/grid-advanced-data-binding.html)or[Declarative DataSources](http://www.telerik.com/help/aspnet-ajax/grid-binding-to-declarative-datasource-controls.html)).
>


Although [simple DataBinding](http://www.telerik.com/help/aspnet-ajax/grid-simple-data-binding.html) is not supported when __ViewState__ is disabled, there is a work-around that could be used in some cases. You need to call DataBind() method on the Grid instance on __Page_Load__ event when the Page is initially loaded and on __Page_Init__ event on any subsequent postback.



````ASPNET
	        <telerik:RadGrid ID="RadGrid1" runat="server" AllowPaging="True" CellSpacing="0"
	            EnableViewState="false" GridLines="None">
	        </telerik:RadGrid>
	        <asp:Button Text="Postback" ID="Button1" runat="server" />
````
````C#
	public partial class Default_Cs : System.Web.UI.Page
	{
	    protected void Page_Load(object sender, EventArgs e)
	    {
	        if (!Page.IsPostBack)
	        {
	            RadGrid1.DataSource = MasterData();
	            RadGrid1.DataBind();
	        }
	
	    }
	
	    protected void Page_Init(object sender, EventArgs e)
	    {
	        if (Page.IsPostBack)
	        {
	            RadGrid1.DataSource = MasterData();
	            RadGrid1.DataBind();
	        }
	    }
	
	    public List<MasterItem> MasterData()
	    {
	        List<MasterItem> masterItems = new List<MasterItem>();
	        for (int i = 0; i < 6; i++)
	        {
	            masterItems.Add(new MasterItem(i, "Master_Value_" + i.ToString()));
	        }
	        return masterItems;
	    }
	}
	
	public class MasterItem
	{
	    public int MasterItemID { get; set; }
	    public string MasterItemValue { get; set; }
	    public MasterItem(int masterItemID, string masterItemValue)
	    {
	        MasterItemID = masterItemID;
	        MasterItemValue = masterItemValue;
	    }
	}
````
````VB.NET
	Partial Class Default_VB
	    Inherits System.Web.UI.Page
	
	    Protected Sub Page_Load(sender As Object, e As EventArgs) Handles Me.Load
	        If Not Page.IsPostBack Then
	            RadGrid1.DataSource = MasterData()
	            RadGrid1.DataBind()
	        End If
	
	    End Sub
	
	    Protected Sub Page_Init(sender As Object, e As EventArgs) Handles Me.Init
	        If Page.IsPostBack Then
	            RadGrid1.DataSource = MasterData()
	            RadGrid1.DataBind()
	        End If
	    End Sub
	
	
	    Public Function MasterData() As List(Of MasterItem)
	        Dim masterItems As New List(Of MasterItem)()
	        For i As Integer = 0 To 5
	            masterItems.Add(New MasterItem(i, "Master_Value_" + i.ToString()))
	        Next
	        Return masterItems
	    End Function
	End Class
	
	Public Class MasterItem
	    Public Property MasterItemID() As Integer
	        Get
	            Return m_MasterItemID
	        End Get
	        Set(value As Integer)
	            m_MasterItemID = Value
	        End Set
	    End Property
	    Private m_MasterItemID As Integer
	    Public Property MasterItemValue() As String
	        Get
	            Return m_MasterItemValue
	        End Get
	        Set(value As String)
	            m_MasterItemValue = Value
	        End Set
	    End Property
	    Private m_MasterItemValue As String
	    Public Sub New(masterItemID__1 As Integer, masterItemValue__2 As String)
	        MasterItemID = masterItemID__1
	        MasterItemValue = masterItemValue__2
	    End Sub
	End Class
````


When having __EnableViewState__ set to false, RadGrid will use the control state of the host Page to store only the absolutely necessary bits of data that preserve the current paging, sorting, and filtering state.

Some operations in Telerik RadGrid like data extraction through the __ExtractValuesFromItem__ method, grouping, hierarchical views expand/collapse, custom edit forms (WebUserControl and FormTemplate) require that the __EnableViewState__ is set to __true__. If no data is persisted for items in Telerik RadGrid (__EnableViewState=false__), then the state of items is lost after postback. Generally, data-source persistence optimization should be used if the small size/speed of a Page, showing RadGrid, is crucial for the application. If ViewState optimization is enabled, RadGrid will fire __NeedDataSource__ and will bind after each post back to restore its items.__RadGrid__ and its TableView manage the state of the following features while the__EnableViewState__ property is set to __False__:

* Indexes of selected items. Because of design limitations RadGrid only persists the selected items on the first postback.Any subsequent postbacks will clear the SelectedIndexes collection and thus will deselect the items. Possible work-around is to call the saveClientState() function of RadGridwhich will save the selected items in the client-state of the control. The same rule applies for selected cells.

* Indexes of edited items

* Sort expressions

* Style properties (but not if any style is applied on a single cell or row in ItemDataBound event)

* Columns order and other column properties

* All settings concerning hierarchy structure (but not the expanded state of the items)

* Filter expressions(but not the filter value in the input control)

* Current page index and the page size

__RadGrid__ does NOT manage the state for the following features when its __EnableViewState__ property is set to __false__:

* Custom edit forms (user control or form template)

* Value of the filter input control, however, the filter expression is persisted and so the filtered results.

* GroupByExpressions are not persisted on Rebind and postback.

* Expanded state of items in hierarchy. This basically means that you could not use more than one level of hierarchy, unlesss you save the expanded settings manually.

If you would like to retain the expanded state of items or server-side selection with __ViewState__ disabled, you may consider using the approach depicted in [this code library entry](http://www.telerik.com/community/code-library/aspnet-ajax/grid/retain-expanded-selected-state-in-hierarchy-on-rebind.aspx) (applicable for explicit rebinds or __ViewState__ switched off) or [this knowledge base article](http://www.telerik.com/support/kb/aspnet-ajax/grid/using-nopersistence-in-datasourcepersistancemode-of-gridtableview.aspx) . An alternative solution would be to turn on the __ViewState__ of the grid and optimize its performance by using __RadCompression__ and its page adapters as explained in [this article](http://www.telerik.com/help/aspnet-ajax/radcompression.html).