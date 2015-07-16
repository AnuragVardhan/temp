function JSDataGrid(parentElementId,id,width,height,columns,title,enableMouseHover,enableMultiSelection)
{
	//public properties
	this.parentElementId = parentElementId;
	this.id = id;
	this.width = width;
	this.height = height;
	this.title = title;
	this.enableMouseHover = enableMouseHover;
	this.enableMultiSelection = enableMultiSelection;
	this.parentElement = null;
	this.columns = columns;
	this.rowSelectionHandler = null;
	this.rowUnSelectionHandler = null;
	this.childField = "children";
	this.columnMinWidth = 10;
	//Constants
	this.__CLASS_SORTING_ASC = "sorted_asc";
	this.__CLASS_SORTING_DESC = "sorted_desc";
	this.__OUTER_CONTAINER_ID = "divDataSet";
	this.__TITLE_CONTAINER_ID = "divTitleBar";
	this.__TABLE_HEADER_CONTAINER_ID = "divHeaderContainer";
	this.__TABLE_HEADER_ID = "tblHeader";
	this.__TABLE_BODY_CONTAINER_ID = "divBodyContainer";
	this.__TABLE_BODY_ID = "tblBody";
	
	this.__FOOTER_CONTAINER_ID = "divFooterContainer";
	this.__FOOTER_TABLE_ID = "tblFooter";
	//private variables
	//storing all the dataFields
	this._dataSource = null;
	//contains all the selected Rows
	this._selectedItems = null;
	//contains details of the column to be dragged
	this._dragColumns = null;
	//store the id's added key
	this.generatedKey = null;
	//stores whether mouse is on GridLine
	this._onGridLine = false;
	//cell which is being resized
	this._resizingHeader = null;
	this._resizingHeaderCell = null;
	this._resizingBodyCell = null;
	this._resizeHandler = null;
	
	/******************************************************Start of Public function Section*************************************************************/
	this.init= function()
	{
		if(!this.parentElementId)
		{
			throw new Error("JSDataGrid Error :: ParentElementId cannot be null");
			return;
		}
		this.parentElement = getElement(this.parentElementId);
		if(!this.parentElement)
		{
			throw new Error("JSDataGrid Error :: ParentElement not found");
			return;
		}
		if(!id || id.length === 0)
		{
			this.id = "jsDatagrid";
		}
		this.createComponentID();
		this._dataSource = new Array();
		this._selectedItems = new Array();
		this._dragColumns = new Object();
		this._dragColumns.zIndex = 0;
		this.createDataSet();
	};
	
	this.dataSource= function(source)
	{
		this._dataSource = source;
		this.createHeaderRows();
		this.createBody(this._dataSource);
	};
	
	/******************************************************End of Public function Section*************************************************************/
	/******************************************************Start of Creating DataGrid Components*************************************************************/
	this.createComponentID= function()
	{
		this.generatedKey = getUniqueId();
		//storing the reference of this as binding does not work in function dragEnd 
		//REASON:"Node was not found" at  _dragColumns.tableContainer.removeChild(_dragColumns.clonedTable)
		window[this.id + this.generatedKey] = this;
		this.__CLASS_SORTING_ASC += this.generatedKey;
		this.__CLASS_SORTING_DESC += this.generatedKey;
		this.__OUTER_CONTAINER_ID += this.generatedKey;
		this.__TITLE_CONTAINER_ID += this.generatedKey;
		this.__TABLE_HEADER_CONTAINER_ID += this.generatedKey;
		this.__TABLE_HEADER_ID += this.generatedKey;
		this.__TABLE_BODY_CONTAINER_ID += this.generatedKey;
		this.__TABLE_BODY_ID += this.generatedKey;
		this.__FOOTER_CONTAINER_ID += this.generatedKey;
		this.__FOOTER_TABLE_ID += this.generatedKey;
	};
	
	this.createDataSet= function()
	{
		this.parentElement.innerHTML = "";
		var divOuterContainer = createDiv(this.__OUTER_CONTAINER_ID);
		if(this.width)
		{
			divOuterContainer.style.width = this.width;
		}
		else
		{
			divOuterContainer.style.width = "100%";
		}
		if(this.height)
		{
			divOuterContainer.style.height = this.height;
		}
		this.parentElement.appendChild(divOuterContainer);
		if(this.title && this.title.length > 0)
		{
			var divTitleBar = this.createTitleBar(this.title);
			divOuterContainer.appendChild(divTitleBar);
		}
		this.createTable();
	};
	
	this.createTitleBar= function(title)
	{
		var divTitleBar = createDiv(this.__TITLE_CONTAINER_ID,"dataGridTitleBar");
		//divTitleBar.style.width = "100%";
		divTitleBar.innerHTML = title;
		
		return divTitleBar;
	};
	
	this.createTable= function()
	{
		this.createHeader();
	};
	
	this.createHeader= function()
	{	
		var divOuterContainer = getElement(this.__OUTER_CONTAINER_ID);
		var divHeader = this.getSectionContainer(this.__TABLE_HEADER_CONTAINER_ID,this.__TABLE_HEADER_ID);
		divHeader.style.overflow = "hidden";
		divOuterContainer.appendChild(divHeader);
		var tblData = getElement(this.__TABLE_HEADER_ID);
	    var tBodies = tblData.tBodies;
	    var tblHeader = null;
	    if(tBodies && tBodies.length > 0)
	    {
	    	 tblData.removeChild(tBodies[0]);
	    }
	    tblHeader = document.createElement("tbody");
	    tblData.appendChild(tblHeader);
		
		//this.createHeaderRows();
	};
	
	this.createHeaderRows= function()
	{
		var tblData = getElement(this.__TABLE_HEADER_ID);
		var tblHeader = tblData.tHead;
		var tblHeaderBody = tblData.tBodies[0];
		if(tblHeader)
	    {
			//IE8 gives runtime exception if we do tblHeader.innerHTML = "" hence doing removeChild
			tblHeader.removeChild(tblHeader.firstChild);
	    }
		else
		{
			 tblHeader = tblData.createTHead();
		}
	    var headerRow = tblHeader.insertRow(-1);
	    headerRow.style.height = "auto";
	    var bodyRow = tblHeaderBody.insertRow(-1);
	    for (var colIndex = 0; colIndex < this.columns.length; colIndex++)
	    {
	    	var headerText = " ";
	        var colItem = this.columns[colIndex];
	        if(colItem.hasOwnProperty("headerText") && colItem["headerText"])
	        {
	        	headerText = colItem["headerText"];
	        }
            var headerCell = headerRow.insertCell(-1);
            var bodyCell = bodyRow.insertCell(-1);
            bodyCell.id = "col" + colItem["headerText"];
            
            var cellDiv = createDiv("div" + headerText); 
            bodyCell.appendChild(cellDiv);
            if(colItem.hasOwnProperty("width") && !isNaN(colItem["width"]))
            {
            	headerCell.style.width = Number(colItem["width"]) + "px";
            }
            else
            {
            	headerCell.style.width = "100" + "px";
            }
            headerCell.style.height = "0px";
            cellDiv.innerHTML = "<b>" + headerText + "</b>";
            addStyleClass(bodyCell , "dataGridHeader");
            var item = null;
            if(this._dataSource && this._dataSource.length > 0 && colItem.hasOwnProperty("dataField") && colItem["dataField"] && colItem.hasOwnProperty("sortable") && colItem["sortable"] === true)
            {
                  for(var count = 0; count < this._dataSource.length; count++)
                  {
                       item = this._dataSource[count][colItem["dataField"]];
                       if(item && item != "")
                       {
                           break;
                       }
                  }
                  if(item && item !="")
                  {
                	  bodyCell.sortFunction = this.determineSortFunction(item);
                  }
                  else
                  {
                	  bodyCell.sortFunction = "sortCaseInsensitive";
                  }
            }   
            bodyCell.columnIndex = colIndex;
            
            bodyCell.onclick = this.headerClickHandler.bind(this);
            bodyCell.onmouseover = this.headerMouseOverHandler.bind(this);
            bodyCell.onmouseout = this.headerMouseOutHandler.bind(this);
	    }
	};
	
	this.createBody= function(dataSet)
	{	
		var divOuterContainer = getElement(this.__OUTER_CONTAINER_ID);
		var divBody = this.getSectionContainer(this.__TABLE_BODY_CONTAINER_ID,this.__TABLE_BODY_ID);
		divBody.style.overflow = "auto";
		divOuterContainer.appendChild(divBody);
		this.renderBody(dataSet);
	};
	
	this.renderBody = function(dataSet)
	{
		var tblData = getElement(this.__TABLE_BODY_ID);
	    var tBodies = tblData.tBodies;
	    var tblBody = null;
	    if(tBodies && tBodies.length > 0)
	    {
	    	 tblData.removeChild(tBodies[0]);
	    }
	    tblBody = document.createElement("tbody");
	    tblData.appendChild(tblBody);
	    this.createBodyHeader();
		this.createBodyComponents(tblBody,dataSet,0,0);
		this.alignTables();
		var divBodyContainer = getElement(this.__TABLE_BODY_CONTAINER_ID);
		makeUnselectable(divBodyContainer);
	};
	
	this.createBodyHeader= function()
	{
		var tblData = getElement(this.__TABLE_BODY_ID);
		var tblHeader = tblData.tHead;
		if(tblHeader)
	    {
			//IE8 gives runtime exception if we do tblHeader.innerHTML = "" hence doing removeChild
			tblHeader.removeChild(tblHeader.firstChild);
	    }
		else
		{
			 tblHeader = tblData.createTHead();
		}
	    var headerRow = tblHeader.insertRow(-1);
	    headerRow.style.height = "auto";
	    for (var colIndex = 0; colIndex < this.columns.length; colIndex++)
	    {
	        var colItem = this.columns[colIndex];
            var headerCell = headerRow.insertCell(-1);
            if(colItem.hasOwnProperty("width") && !isNaN(colItem["width"]))
            {
            	headerCell.style.width = Number(colItem["width"]) + "px";
            }
            else
            {
            	headerCell.style.width = "100" + "px";
            }
            headerCell.style.height = "0px";
	    }
	};
	
	this.createBodyComponents= function(tblBody,dataSet,parentIndex,level)
	{
	     if(dataSet)
	     {
	    	 for (var rowIndex = 0; rowIndex < dataSet.length; rowIndex++)
		     {
	    		var item = dataSet[rowIndex];
		        var row = tblBody.insertRow(-1);
		        if(parentIndex > 0)
		        {
		        	row.setAttribute("parent-row-count",parentIndex);
		        }
		        var totalRowCount = this.getTotalRows() - 1;
		        var className = null;
		        if((totalRowCount % 2) === 0)
		        {
		            className = "dataGridEvenRow";
		        }
		        else
		        {
		            className = "dataGridOddRow";
		        }
		        this.createRow(row,item,className,totalRowCount,level);
		        addStyleClass(row , className);
		        row.source = item;
		        row.index = rowIndex + level;
		        row.onmouseover = this.rowMouseHover.bind(this);
		        row.onmouseout = this.rowMouseHover.bind(this);
		        row.onclick =  this.rowClickHandler.bind(this);
		        if(item.hasOwnProperty(this.childField) && item[this.childField]  && item[this.childField].length > 0)
	            {
		        	this.createBodyComponents(tblBody,item[this.childField],totalRowCount,level + 1);
		        	if(item.hasOwnProperty("_isExpanded"))
		        	{
		        		var divChild = getFirstChild(row.cells[0]);
		        		if(divChild)
		        		{
		        			var compArrow  = getFirstChild(divChild);
		        			var isExpanded = hasStyleClass(compArrow,"arrow-down");
		        			this.hideShowRow(totalRowCount,compArrow,true);
		        			if(isExpanded != item["_isExpanded"])
							{
		        				//this.hideShowRow(totalRowCount,compArrow,!item["_isExpanded"]);
							}
		        		}
		        	}
		        	else
		        	{
		        		item["_isExpanded"] = true;
		        	}
		        	this.hideShowRow(totalRowCount,compArrow,true);
	            }
		        //addEvent(trs[i],'click',this.callBodyClick);
		        //addEvent(trs[i],'dblclick',this.callBodyDblClick);
		     }
	     }
	};
	
	this.createRow= function(row,item,className,parentIndex,level)
	{
	    if(item)
	    {
	        for (var colIndex = 0; colIndex < this.columns.length; colIndex++)
	        {
	        	var colItem = this.columns[colIndex];
	        	if((colItem.hasOwnProperty("dataField") && colItem["dataField"]) || (colItem.hasOwnProperty("templateID") && colItem["templateID"])
	        			|| (colItem.hasOwnProperty("itemRenderer") && colItem["itemRenderer"]))
	        	{
	        		var dataField = colItem["dataField"];
	        		var templateID = colItem["templateID"];
		            var cell = row.insertCell(-1);
		            if(className && className.length > 0)
		            {
		                addStyleClass(cell , "dataGridCell");
		            }
		            var cellDiv = createDiv(null);
		            cell.appendChild(cellDiv);
		            if(colIndex == 0)
		            {
		            	if(level == 0)
		            	{
		            		cell.style.paddingLeft = "5px";
		            	}
		            	else
		            	{
		            		cell.style.paddingLeft = (20 * level) + "px";
		            	}
		            	
		            }
		            if(colIndex == 0 && item.hasOwnProperty(this.childField) && item[this.childField]  && item[this.childField].length > 0)
		            {
		            	addStyleClass(cellDiv,"hbox");
		            	var cellArrowParent = createDiv(null);
		            	var compArrow = this.createArrow(parentIndex);
		            	cellArrowParent.appendChild(compArrow);
		            	cellDiv.appendChild(cellArrowParent);
		            	//cellDiv.appendChild(document.createTextNode('\u00A0'));
		            	var cellText = createDiv(null);
		            	this.addCellText(item,cellText,colItem,colIndex);
		            	cellDiv.appendChild(cellText);
		            }
		            else
		            {
		            	this.addCellText(item,cellDiv,colItem,colIndex);
		            }
	        	}
	        }
	    }
	};
	
	this.alignTables = function()
	{
		var divOuterContainer = getElement(this.__OUTER_CONTAINER_ID);
		var divHeader = getElement(this.__TABLE_HEADER_CONTAINER_ID);
		var divBody = getElement(this.__TABLE_BODY_CONTAINER_ID);
		var divTitleBar = getElement(this.__TITLE_CONTAINER_ID);
		var topHeight = divHeader.offsetHeight;
		if(divTitleBar)
		{
			topHeight += divTitleBar.offsetHeight;
		}
		divBody.style.height = (divOuterContainer.offsetHeight - topHeight) + "px";
		var tableHeader = getElement(this.__TABLE_HEADER_ID);
		var tableBody = getElement(this.__TABLE_BODY_ID);
		var scrollBarWidth = getScrollBarWidth(divBody);
		divHeader.style.width = (divBody.offsetWidth - scrollBarWidth) + "px";
		divBody.onscroll = this.synchronizeTables.bind(this);
		
		if(tableHeader.tHead && tableHeader.tHead.rows.length > 0 && tableBody.tHead && tableBody.tHead.rows.length)
		{
			var headerCells = tableHeader.tHead.rows[0].cells;
			var bodyCells = tableBody.tHead.rows[0].cells;
			var totalWidth = 0;
			if(headerCells && headerCells.length > 0 && bodyCells && bodyCells.length > 0)
			{
				for(var count = 0;count < headerCells.length;count++)
				{
					totalWidth += bodyCells[count].offsetWidth;
					headerCells[count].style.width = bodyCells[count].offsetWidth + "px";
					bodyCells[count].style.width = bodyCells[count].offsetWidth + "px";
				}
				tableBody.style.width = totalWidth + "px";
				tableHeader.style.width = totalWidth + "px";
			}
		}
		
		//tableHeader.style.tableLayout = "fixed";
		//tableBody.style.tableLayout = "fixed";
	};
	
	this.synchronizeTables = function(event) 
	{
		var divHeader = getElement(this.__TABLE_HEADER_CONTAINER_ID);
		//divHeader.scrollTop = event.srcElement.scrollTop;
		divHeader.scrollLeft = event.srcElement.scrollLeft;
	};
	
	this.getSectionContainer = function(containerID,tableID)
	{
		var divTableContainer = createDiv(containerID); 
		var table = document.createElement("TABLE");
		table.id = tableID;
		addStyleClass(table , "dataGrid");
		//addStyleClass(tblDataSet , "draggable");
		addStyleClass(table , "resizable");
		divTableContainer.appendChild(table);
		return divTableContainer;
	};
	
	
	this.createArrow= function(parentRowCount)
	{
		 var compArrow = createDiv("compArrow" + parentRowCount,"arrow-down");
		 compArrow.setAttribute("parent-row-count",parentRowCount);
		 compArrow.onclick =  this.arrowClickHandler.bind(this);
		 return compArrow;
	};
	
	this.addCellText = function(item,div,colItem,colIndex)
	{
		var text = "";
		var dataField = colItem["dataField"];
		var templateID = colItem["templateID"];
		var itemRenderer = colItem["itemRenderer"];
		if(itemRenderer)
		{
			var strRenderer = itemRenderer(item,dataField,this.getTotalRows() - 1,colIndex);
			if(strRenderer)
			{
				var compBodySpan = document.createElement("span");
				compBodySpan.innerHTML = strRenderer;
				div.appendChild(compBodySpan);
				return ;
			}
		}
		if(templateID)
		{
			var template = getTemplate(templateID);
			if(template)
			{
				div.appendChild(template);
				return ;
			}
		}
		if(item && item.hasOwnProperty(dataField) && item[dataField])
        {
			text = item[dataField];
        }
		div.appendChild(document.createTextNode(text));
	};
	
	this.isMouseOnElement = function(element, currentX, currentY)
	{
		var offset = getCumulativeOffset(element);
		return (currentY >= offset.y &&
				currentY <  offset.y + element.offsetHeight &&
	            currentX >= offset.x + element.offsetWidth - 5 &&
	            currentX <  offset.x + element.offsetWidth);
	};
	
	this.arrowClickHandler = function(event)
	{
		var target = event.target;
		if(target && target.hasAttribute("parent-row-count"))
		{
			var rowNum = target.getAttribute("parent-row-count");
			var isCollapse = true;
			if(target.className == "arrow-right")
			{
				isCollapse = false;
			}
			this.hideShowRow(rowNum,target,isCollapse);
		}
	};
	
	this.hideShowRow = function(rowCount,compArrow,isCollapse)
	{
		if(rowCount > 0)
		{
			var arrChildRows = [];
			this.getChildRows(arrChildRows,rowCount,isCollapse);
			if(arrChildRows && arrChildRows.length > 0)
			{
				for(var count = 0;count < arrChildRows.length;count++)
				{
					var row = arrChildRows[count];
					if(isCollapse)
					{
						row.style.display = "none";
						if(row && row.hasAttribute("parent-row-count"))
						{
							var rowParentCount = row.getAttribute("parent-row-count");
							if(rowParentCount)
							{
								var divArrow = this.getArrows(rowParentCount);
								if(divArrow)
								{
									divArrow.className = "arrow-right";
								}
							}
						}
					}
					else
					{
						row.style.display = "";
					}
				}
				if(compArrow)
				{
					if(isCollapse)
					{
						compArrow.className = "arrow-right";
					}
					else
					{
						compArrow.className = "arrow-down";
					}
				}
			}
		}
	};
	
	this.getChildRows = function(sendRows,rowCount,includeAllChildren)
	{
		var tblData = getElement(this.__TABLE_BODY_ID);
		var arrRows = tblData.querySelectorAll("tr");
		if(arrRows && arrRows.length > 0)
		{
			for(var count = 0;count < arrRows.length;count++)
			{
				var row = arrRows[count];
				if(row && row.hasAttribute("parent-row-count"))
				{
					var rowParentCount = row.getAttribute("parent-row-count");
					if(rowParentCount)
					{
						if(includeAllChildren)
						{
							if(rowParentCount == rowCount)
							{
								sendRows[sendRows.length] = row;
								this.getChildRows(sendRows,count,includeAllChildren);
							}
						}
						else if(rowParentCount == rowCount)
						{
							sendRows[sendRows.length] = row;
						}
					}
				}
			}
		}
	};
	
	this.getArrows = function(rowCount)
	{
		var tblData = getElement(this.__TABLE_BODY_ID);
		var arrDivs = tblData.querySelectorAll("div");
		if(arrDivs && arrDivs.length > 0)
		{
			for(var count = 0;count < arrDivs.length;count++)
			{
				var div = arrDivs[count];
				if(div && div.hasAttribute("parent-row-count"))
				{
					var rowParentCount = div.getAttribute("parent-row-count");
					if(rowParentCount && rowParentCount == rowCount)
					{
						return div;
					}
				}
			}
		}
	};
	
	this.getTotalRows = function()
	{
		var tblData = getElement(this.__TABLE_BODY_ID);
		if(tblData)
		{
			return tblData.rows.length;
		}
		return 0;
	};
	//http://msdn.microsoft.com/en-us/library/ms532998%28v=vs.85%29.aspx
	this.createFooter= function(table)
	{
	    var oRow, oCell;
	    var oTFoot = table.createTFoot();
	    var oCaption = table.createCaption();
	    oRow = oTFoot.insertRow();
	    oCell = oRow.insertCell();
	      oCell.innerHTML = "Quotes are for example only.";
	     oCell.colSpan = "4";
	    oCell.bgColor = "lightskyblue";
	 
	      // Set the innerText of the caption and position it at the bottom of the table.
	    oCaption.innerHTML = "Created using Table Object Model.";
	    oCaption.style.fontSize = "10px";
	    oCaption.align = "bottom";
	};
	
	this.addAscendingIndicator= function(target)
	{
	     if(target)
	     {
	          var indicator_Asc = document.createElement("span");
	          indicator_Asc.id = "indicator_Asc";
	          indicator_Asc.innerHTML = isBrowserIE() ? "&nbsp<font face='webdings'>5</font>" : "&nbsp;&#x25B4;";
	          //add the span in the div of the TR
	          target.firstChild.appendChild(indicator_Asc);
	          addStyleClass(target,this.__CLASS_SORTING_ASC);
	     }
	};
	
	this.removeAscendingIndicator= function(target)
	{
	     if(target)
	     {
	          removeStyleClass(target,this.__CLASS_SORTING_ASC);
	          var indicator_Asc = getElement("indicator_Asc");
	          if(indicator_Asc && indicator_Asc.parentNode)
	          {
	        	  indicator_Asc.parentNode.removeChild(indicator_Asc);
	          }
	     }
	};
	
	this.addDescendingIndicator= function(target)
	{
	    if(target)
	    {
	         var indicatorDesc = document.createElement("span");
	         indicatorDesc.id = "indicator_Desc";
	         indicatorDesc.innerHTML = isBrowserIE() ? "&nbsp<font face='webdings'>6</font>" : "&nbsp;&#x25BE;";
	         //add the span in the div of the TR
	         target.firstChild.appendChild(indicatorDesc);
	         addStyleClass(target,this.__CLASS_SORTING_DESC);
	    }
	};
	
	this.removeDescendingIndicators= function(target)
	{
	     if(target)
	     {
	          removeStyleClass(target,this.__CLASS_SORTING_DESC);
	          var indicator_Desc = getElement("indicator_Desc");
	          if(indicator_Desc && indicator_Desc.parentNode)
	          {
	        	  indicator_Desc.parentNode.removeChild(indicator_Desc);
	          }
	     }
	};
	
	this.resetIndicators= function(target)
	{
	     if(target)
	     {
	          this.removeAscendingIndicator(target);
	          this.removeDescendingIndicators(target);
	     }
	};
	
	this.resetColumnHeaders= function()
	{
	     var tblData = getElement(this.__TABLE_HEADER_ID);
	     var tblHeaderBody = null;
	     //safari doesnot support table.tHead
	     if (tblData.tBodies && tblData.tBodies.length > 0)
	     {
	    	 tblHeaderBody = tblData.tBodies[0];
	     }
	     //two header not allowed
	     if (tblHeaderBody.rows.length != 1)
	     {
	          return;
	     }
	     var headers = tblHeaderBody.rows[0].cells;
	     for (var colCount = 0; colCount < headers.length; colCount++)
	     {
	          this.resetIndicators(headers[colCount]);
	     }
	};
	/******************************************************End of Creating DataGrid Components*************************************************************/
	/******************************************************Start of Event Handlers*************************************************************/
	this.headerClickHandler= function(event)
	{
		 if(this._onGridLine)
		{
			 return;
		}
	     var target = getTarget(event);
	     //adding the below condition so that if we add a span or a font and click on it then we should navigate till we find the header object
	     target = findParent(target,"TD");
	     var columnDetail = this.columns[target.columnIndex];
	     if(columnDetail && columnDetail.sortable)
	     {
	            // if last sorted column and current sorted column are same just reverse the dataset
	          /*if (hasStyleClass(target,this.__CLASS_SORTING_ASC) || hasStyleClass(target,this.__CLASS_SORTING_DESC))
	          {
	               this.reverseTable();
	               if(hasStyleClass(target,this.__CLASS_SORTING_ASC))
	               {
	                    this.removeAscendingIndicator(target);
	                    this.addDescendingIndicator(target);
	               }
	               else if(hasStyleClass(target,this.__CLASS_SORTING_DESC))
	               {
	            	   this.removeDescendingIndicators(target);
	            	   this.addAscendingIndicator(target);
	               }
	               return;
	          }*/
	    	  var sortAscending = false;
	    	  if (hasStyleClass(target,this.__CLASS_SORTING_ASC) || hasStyleClass(target,this.__CLASS_SORTING_DESC))
	          {
	               if(hasStyleClass(target,this.__CLASS_SORTING_ASC))
	               {
	                    this.removeAscendingIndicator(target);
	                    sortAscending = false;
	               }
	               else if(hasStyleClass(target,this.__CLASS_SORTING_DESC))
	               {
	            	   this.removeDescendingIndicators(target);
	            	   sortAscending = true;
	               }
	          }
	    	  else
	    	  {
	    		  this.resetColumnHeaders();
	    		  sortAscending = !columnDetail.sortDescending;
	    	  }
	    	  if(sortAscending)
	          {
	    		  this.addAscendingIndicator(target);
	          }
	          else
	          {
	        	  this.addDescendingIndicator(target);
	          }
	          var dataSorted = this._dataSource.slice(0);
	          this.sortArrOfObjectsByParam(this._dataSource,target.sortFunction,this.columns[target.columnIndex].dataField,sortAscending);
	          this.renderBody(this._dataSource);
	     }
	};
	
	this.headerMouseOverHandler = function(event)
	{
		event = getEvent(event);
		var target = getTarget(event);
		target = findParent(target,"TD");
		target.onmousemove = this.headerMouseMoveHandler.bind(this);
		target.onmousedown = this.headerMouseDownHandler.bind(this);
	};
	
	this.headerMouseOutHandler = function(event)
	{
		event = getEvent(event);
		var target = getTarget(event);
		target = findParent(target,"TD");
		this._onGridLine = false;
		
		target.onmousemove = null;
		target.onmousedown = null;
		
		removeStyleClass(target,"resize-handle-active");
	};
	
	this.headerMouseMoveHandler = function(event)
	{
		event = getEvent(event);
		var target = getTarget(event);
		target = findParent(target,"TD");
		if(this.isMouseOnElement(target,event.clientX,event.clientY))
		{
			addStyleClass(target,"resize-handle-active");
			this._onGridLine = true;
		}
		else
		{
			removeStyleClass(target,"resize-handle-active");
			this._onGridLine = false;
		}
	};
	
	this.headerMouseDownHandler = function(event)
	{
		event = getEvent(event);
	    var target = getTarget(event);
	    target = findParent(target,"TD");
	    this.startResize(event,target);
	    //this.dragStart(event,target);
	};
	
	this.rowMouseHover= function(event)
	{
		 if(this.enableMouseHover)
		 {
			 event = getEvent(event);
		     var target = getTarget(event);
		     target = findParent(target,"TR");
		     if (event.type == "mouseover")
		     {
		         addStyleClass(target,"dataGridHover");
		     }
		     else if (event.type == "mouseout")
		     {
		         removeStyleClass(target,"dataGridHover");
		     }
		 }
		    return false;
	};
	
	this.rowClickHandler= function(event)
	{
		event = getEvent(event);
	    var target = getTarget(event);
	    target = findParent(target,"TR");
	    //Multiselection Check
	    if (event.shiftKey && this.enableMultiSelection)
	    {
	    	this.multiSectionHandler(target);
	    }
	    else if(event.ctrlKey && this.enableMultiSelection)
	    {
	      if(this.isRowSelected(target))
	      {
	    	  this.markRowUnselected(target);
	      }
	      else
	      {
	    	  this.markRowSelected(target);
	      }
	    }
	    else
	    {
	    	this.clearAllRowSelection();
	    	this.markRowSelected(target);
	    } 
	};
	
	this.resize = function(table,cell, desirableWidth)
	{
		var cellIndex = cell.cellIndex;
		var cellPaddingLeft = getDimensionAsNumber(cell,"paddingLeft");
		var cellPaddingRight = getDimensionAsNumber(cell,"paddingRight");
		var pad = parseInt(cellPaddingLeft,10) + parseInt(cellPaddingRight,10);
		var setWidth = Math.max(desirableWidth - pad, this.columnMinWidth);
		console.log(setWidth);
		
		cell.style.width = setWidth + "px";
	};
	
	this.startResize = function(event,target)
	{
		if(!this._onGridLine)
		{
			return;
		}
		
		target.onmousemove = null;
		target.mousedown = null;
		target.mouseout = null;
		target.onclick = null;
		
		this._resizingHeader = target;
		this._resizingHeaderCell = this.getHeaderTopCell(target);
		this._resizingBodyCell = this.getBodyTopCell(target);
		
		var headerTable = getElement(this.__TABLE_HEADER_ID);
		
		this.createResizeHandler(event);
		
		document.onmousemove = this.doResize.bind(this);
		document.onmouseup = this.endResize.bind(this);
		
		event.stopImmediatePropagation();
	};
	
	this.doResize = function(event)
	{
		//putting this check as somehow the document evenlistener is not getting detached
		if(!this._resizingHeader)
		{
			return;
		}
		event = getEvent(event);
		var headerTable = getElement(this.__TABLE_HEADER_ID);
		var tableBody = getElement(this.__TABLE_BODY_ID);
		var selectedHeader = this._resizingHeader;
		var desiredWidth = event.clientX - getCumulativeOffset(selectedHeader).x;
		//this.resize(headerTable,this._resizingHeaderCell,desiredWidth);
		//this.resize(headerTable,this._resizingHeaderCell,desiredWidth);
		
		this._resizeHandler.style.left = event.clientX + "px";
		
		return false;
	};
	
	this.endResize = function(event)
	{
		//putting this check as somehow the document evenlistener is not getting detached
		if(!this._resizingHeader)
		{
			return;
		}
		event = getEvent(event);
		var selectedHeader = this._resizingHeader;
		var tableHeader = getElement(this.__TABLE_HEADER_ID);
		var tableBody = getElement(this.__TABLE_BODY_ID);
		var desiredWidth = event.clientX - getCumulativeOffset(selectedHeader).x;
		//this.resize(tableHeader,this._resizingHeaderCell,desiredWidth);
		this.resize(tableBody,this._resizingBodyCell,desiredWidth);
		
		document.onmousemove = null;
		document.onmouseup = null;
		
		if(this._resizeHandler)
		{
			document.body.removeChild(this._resizeHandler);
		}
		
		this._resizingHeader = null;
		this._resizingHeaderCell = null;
		this._resizingBodyCell = null;
		this._resizeHandler = null;
		event.stopImmediatePropagation();
		this.alignTables();
		selectedHeader.onmouseout = this.headerMouseOutHandler.bind(this);
		selectedHeader.onclick = this.headerClickHandler.bind(this);
	};
	
	this.createResizeHandler = function(event)
	{
		this._resizeHandler = createDiv(null,"resize-handle");
		this._resizeHandler.style.top = getCumulativeOffset(this._resizingHeader).y + "px";
		this._resizeHandler.style.left = event.clientX + "px";
		var divHeader = getElement(this.__TABLE_HEADER_CONTAINER_ID);
		var divBody = getElement(this.__TABLE_BODY_CONTAINER_ID);
		var scrollBarWidth = getScrollBarWidth(divBody);
		this._resizeHandler.style.height = (divHeader.offsetHeight + divBody.offsetHeight - scrollBarWidth) + "px";
		
		document.body.appendChild(this._resizeHandler);
	}
	
	this.getHeaderTopCell = function(cell)
	{
		if(cell)
		{
			var index = cell.cellIndex;
			var tableHeader = getElement(this.__TABLE_HEADER_ID);
			var headerCells = tableHeader.tHead.rows[0].cells;
			return headerCells[index];
		}
		return null;
	};
	
	this.getBodyTopCell = function(cell)
	{
		if(cell)
		{
			var index = cell.cellIndex;
			var tableBody = getElement(this.__TABLE_BODY_ID);
			var bodyCells = tableBody.tHead.rows[0].cells;
			return bodyCells[index];
		}
		return null;
	};
	
	/*callBodyDblClick:function(e) {
	    var elm = SortedTable.getEventElement(e);
	    var st = SortedTable.getSortedTable(elm);
	    if (!st) return false;
	    if (typeof(st.onbodydblclick)=='function') st.onbodydblclick(elm,e);
	    return false;
	},*/
	/******************************************************End of Event Handlers*************************************************************/
	/******************************************************Start of Selection Functions*************************************************************/
	 
	this.isRowSelected= function(row)
	{
	    if(row)
	    {
	        return hasStyleClass(row,"dataGridSelection");
	    }   
	    return false;
	};
	
	this.markRowSelected= function(row)
	{
	    if(row)
	    {
	        if(!this.isRowSelected(row))
	        {
	            addStyleClass(row,"dataGridSelection");   
	            this._selectedItems.push(row);
	            if(isFunction(this.rowSelectionHandler))
	            {
	                this.rowSelectionHandler(row.source);
	            }
	        }
	    }
	};
	
	this.markRowUnselected= function(row)
	{
	    if(this.isRowSelected(row))
	    {
	        removeStyleClass(row,"dataGridSelection");
	        for (var count=0; count < this._selectedItems.length ; count++)
	        {
	            if (this._selectedItems[count].index === row.index)
	            {
	                this._selectedItems.splice(count,1);
	                break;
	            }
	        }
	        if(isFunction(this.rowUnSelectionHandler))
	        {
	            this.rowUnSelectionHandler(row.source);
	        }
	    }
	};
	
	this.clearAllRowSelection= function()
	{
	    for (var count=0; count < this._selectedItems.length ; count++)
	    {
	        if (this._selectedItems[count])
	        {
	        	removeStyleClass(this._selectedItems[count],"dataGridSelection");
	        }
	    }
	    this._selectedItems = new Array();
	};
	
	this.multiSectionHandler= function(lastRow)
	{
		 if(!lastRow)
		 {
			 return;
		 }
		 if (this._selectedItems.length === 0)
		 {
			 this.isRowSelected(lastRow);
		     return;
		 }
		 var firstRow = this._selectedItems[this._selectedItems.length - 1];
		 if(lastRow.index === firstRow.index)
		 {
			 this.markRowUnselected(lastRow);
			 return;
		 }
		 var isDown = (lastRow.index > firstRow.index);
		 var isSelection = !this.isRowSelected(lastRow);
		 var navigateRow = firstRow;
		 do
		 {
			  navigateRow = isDown ? navigateRow.nextSibling : navigateRow.previousSibling;
			  if (isSelection)
			  {
				  this.markRowSelected(navigateRow);
			  }
			  else
			  {
				  this.markRowUnselected(navigateRow);
			  }
		 }
		 while(navigateRow.index != lastRow.index);
	};
	
	this.setVisibilityOfColumn = function(index,isVisible)
	{
		var style;
	    if (isVisible) 
	    {
	    	style = "";
	    }
	    else
	    {
	    	style = "none";
	    }
		
		var tableHeader = getElement(this.__TABLE_HEADER_ID);
		var tableBody = getElement(this.__TABLE_BODY_ID);
		var rows = tableHeader.getElementsByTagName("tr");
		for (var count = 0; count < rows.length;count++) 
		{
		     var cells = rows[count].getElementsByTagName("td");
		     if(cells.length > index)
		     {
		    	 cells[index].style.display = style;
		     }
		     else
		     {
		    	 return;
		     }
		}
		rows = tableBody.getElementsByTagName("tr");
		for (var count = 0; count < rows.length;count++) 
		{
		     var cells = rows[count].getElementsByTagName("td");
		     cells[index].style.display = style;
		}
		
	};
	/******************************************************End of Selection Functions*************************************************************/
	/******************************************************Start of Sorting Logic*************************************************************/
	//This method is based on Stuart Langridge's "sorttable" code
	this.determineSortFunction= function(item)
	{ 
	      var sortFunction = "sortCaseInsensitive";
	     
	      if (item.match(/^\d\d[\/-]\d\d[\/-]\d\d\d\dgetElement/))
	      {
	          sortFunction = "sortDate";
	      }
	      if (item.match(/^\d\d[\/-]\d\d[\/-]\d\dgetElement/))
	      {
	          sortFunction = "sortDate";
	      }
	      if (item.match(/^[£$]/))
	      {
	          sortFunction = "sortCurrency";
	      }
	      if (item.match(/^\d?\.?\d+getElement/))
	      {
	             sortFunction = "sortNumeric";
	      }
	      if (item.match(/^[+-]?\d*\.?\d+([eE]-?\d+)?getElement/))
	      {
	             sortFunction = "sortNumeric";
	      }
	     
	      return sortFunction;
	};

	this.sortArrOfObjectsByParam= function(arrToSort,sortFunctionName,dataField,sortAscending)
	{
	     if(sortAscending == null || sortAscending == undefined)
	     {
	         sortAscending = true;  // default to true
	     }
	     arrToSort.sort(function (item1, item2)
	     {
	         var retValue = 0;
	         if (typeof this[sortFunctionName] === "function")
	         {
	             retValue = this[sortFunctionName](item1, item2 , dataField, sortAscending);
	         }
	         return retValue;
	     }.bind(this));
	};
	 
	this.sortCaseInsensitive= function(item1, item2 , dataField, sortAscending)
	{
		if(!item1[dataField] && !item2[dataField])
		{
			return 0;
		}
		var retValue = -1;
		if(!item1[dataField])
		{
			retValue = -1;
		}
		else if(!item2[dataField])
		{
			retValue = 1;
		}
		else
		{
			var firstString = item1[dataField].toLowerCase();
		    var secondString = item2[dataField].toLowerCase();
		      
		    if(firstString == secondString)
		    {
		    	return 0;
		    }
		    if (firstString < secondString)
		    {
		        retValue = -1;
		    }
		    else
		    {
		        retValue = 1;
		    }
		}
		
	    if(sortAscending)
	    {
	        return retValue; 
	    }
	    return (retValue * -1);
	};
	 
	this.sortDate= function(item1, item2 , dataField, sortAscending)
	{
	      // y2k notes: two digit years less than 50 are treated as 20XX, greater than 50 are treated as 19XX
	      var firstDateString = item1[dataField];
	      var secondDateString = item2[dataField];
	      var firstDate, secondDate, year = -1;
	     
	      if (firstDateString.length == 10)
	      {
	               firstDate = firstDateString.substr(6,4) + firstDateString.substr(3,2) + firstDateString.substr(0,2);
	      }
	      else
	      {
	              year = firstDateString.substr(6,2);
	           if (parseInt(year) < 50)
	           {
	                year = "20" + year;
	           }
	           else
	           {
	                year = "19" + year;
	           }
	              firstDate = year + firstDateString.substr(3,2) + firstDateString.substr(0,2);
	      }
	     
	      if (secondDateString.length == 10)
	      {
	           secondDate = secondDateString.substr(6,4)+secondDateString.substr(3,2)+secondDateString.substr(0,2);
	      }
	      else
	      {
	           year = secondDateString.substr(6,2);
	           if (parseInt(year) < 50)
	           {
	                year = "20" + year;
	           }
	           else
	           {
	                year = "19" + year;
	           }
	           secondDate = year + secondDateString.substr(3,2) + secondDateString.substr(0,2);
	      }
	     
	      if (firstDate == secondDate)
	      {
	               return 0;
	      }
	      var retValue = -1;
	      if (firstDate < secondDate)
	      {
	          retValue = -1;
	      }
	      else
	      {
	          retValue = 1;
	      }
	      if(sortAscending)
	      {
	          return retValue; 
	      }
	      return (retValue * -1);
	};
	
	this.sortCurrency= function(item1, item2 , dataField, sortAscending)
	{
	      var firstCurrency = item1[dataField].replace(/[^0-9.]/g,"");
	      var secondCurrency = item2[dataField].replace(/[^0-9.]/g,"");
	      if(sortAscending)
	      {
	          return parseFloat(firstCurrency) - parseFloat(secondCurrency); 
	      }
	      return parseFloat(secondCurrency) - parseFloat(firstCurrency);
	};
	
	this.sortNumeric= function(item1, item2 , dataField, sortAscending)
	{
	      var firstNumber = parseFloat(item1[dataField]);
	      if (isNaN(firstNumber))
	      {
	          firstNumber = 0;
	      }
	      var secondNumber = parseFloat(item2[dataField]);
	      if (isNaN(secondNumber))
	      {
	          secondNumber = 0;
	      }
	      if(sortAscending)
	      {
	          return (firstNumber - secondNumber);
	      }
	      return (secondNumber - firstNumber);
	};
	
	this.reverseTable= function()
	{
	    // reverse the rows in a tbody
	    var tbody = getElement(this.__TABLE_BODY_ID).tBodies[0];
	    var indexCount = 0;
	    newrows = new Array();
	    for (var rowCount=0; rowCount < tbody.rows.length; rowCount++)
	    {
	        newrows[newrows.length] = tbody.rows[rowCount];
	    }
	    for (var rowCount=newrows.length-1; rowCount >= 0; rowCount--)
	    {
	    	var row = newrows[rowCount];
	    	row.index = indexCount;
	    	indexCount ++;
	        tbody.appendChild(row);
	    }
	    delete newrows;
	};
	/******************************************************End of Sorting Logic*************************************************************/
	/******************************************************Start of Dragging Logic*************************************************************/
	this.dragStart= function(event,target) 
	{
	    var _dragColumns = this._dragColumns;
	    
	    var targetPosition = getPositionFromEvent(event);
	    _dragColumns.target = target;

	    // Since a column header can't be dragged directly, copy its contents in a div.
	    var table = getElement(this.__TABLE_ID);
	    this._dragColumns.table = table;
	    this._dragColumns.tableContainer = getElement(this.__TABLE_CONTAINER_ID);
	    this._dragColumns.startColumn = this.getColumnByPostion(table, targetPosition.x);
	    if (this._dragColumns.startColumn == -1) 
	    {
	    	return;
	    }
	    var clonedTable = cloneElement(table, false);
	    clonedTable.style.margin = "0";
	    // Copy the heading
	    if (table.tHead) 
	    {
	    	clonedTable.appendChild(this.copyColumnSection(table.tHead, this._dragColumns.startColumn));
	    }

	    var targetAbsolutePosition = getAbsolutePosition(this._dragColumns.target, true);
	    clonedTable.style.position = "absolute";
	    clonedTable.style.left = targetAbsolutePosition.x + "px";
	    clonedTable.style.top = targetAbsolutePosition.y + "px";
	    clonedTable.style.width = _dragColumns.target.offsetWidth + "px";
	    clonedTable.style.height = _dragColumns.target.offsetHeight + "px";
	    clonedTable.style.opacity = 0.7;

	    // Hold off adding the element until this is clearly a drag.
	    this._dragColumns.addedNode = false;
	    this._dragColumns.clonedTable = clonedTable;

	    // Save starting positions of cursor and element.
	    this._dragColumns.cursorStartX = targetPosition.x;
	    this._dragColumns.cursorStartY = targetPosition.y;
	    this._dragColumns.clonedTableStartLeft  = parseInt(_dragColumns.clonedTable.style.left, 10);
	    this._dragColumns.clonedTableStartTop   = parseInt(_dragColumns.clonedTable.style.top,  10);

	    if (isNaN(_dragColumns.clonedTableStartLeft)) 
	    {
	    	_dragColumns.clonedTableStartLeft = 0;
	    }
	    if (isNaN(_dragColumns.clonedTableStartTop))
	    {
	    	_dragColumns.clonedTableStartTop  = 0;
	    }

	    // Update element's z-index.
	    this._dragColumns.clonedTable.style.zIndex = ++_dragColumns.zIndex;

	    // Capture mousemove and mouseup events on the page.
	    //storing the reference of this as binding does not work in function dragEnd 
		//REASON:"Node was not found" at  _dragColumns.tableContainer.removeChild(_dragColumns.clonedTable)
	    window.jsDataGridId = this.id;
	    window.jsDataGridKey = this.generatedKey;
	    if (isBrowserIE()) 
	    {
	    	document.attachEvent("onmousemove", this.dragMove.bind(this));
	    	document.attachEvent("onmouseup", this.dragEnd);
	    	window.event.cancelBubble = true;
	    	window.event.returnValue = false;
	    } 
	    else 
	    {
	    	document.addEventListener("mousemove", this.dragMove.bind(this), true);
	    	document.addEventListener("mouseup", this.dragEnd, true);
	    	event.preventDefault();
	    }
	};
	
	this.dragMove= function(event) 
	{
	    event = getEvent(event);
	    var _dragColumns = this._dragColumns;

	    // Get cursor position with respect to the page.
	    var targetPosition = getPositionFromEvent(event);

	    var dx = this._dragColumns.cursorStartX - targetPosition.x;
	    var dy = this._dragColumns.cursorStartY - targetPosition.y;
	    if (!_dragColumns.addedNode && dx * dx + dy * dy > 100) 
	    {
	    	_dragColumns.tableContainer.insertBefore(_dragColumns.clonedTable,_dragColumns.table);
	    	_dragColumns.addedNode = true;
	    }

	    // Move drag element by the same amount the cursor has moved.
	    var style = _dragColumns.clonedTable.style;
	    style.left = (_dragColumns.clonedTableStartLeft + targetPosition.x - _dragColumns.cursorStartX) + "px";
	    style.top  = (_dragColumns.clonedTableStartTop  + targetPosition.y - _dragColumns.cursorStartY) + "px";
	 };

	 //if we use bind then it gives "Node was not found" at  _dragColumns.tableContainer.removeChild(_dragColumns.clonedTable);
	  this.dragEnd= function(event) 
	  {
		    var refJSDataGrid = window[window.jsDataGridId +  window.jsDataGridKey];
			if (isBrowserIE()) 
			{
				document.detachEvent("onmousemove", refJSDataGrid.dragMove.bind(refJSDataGrid));
				document.detachEvent("onmouseup", refJSDataGrid.dragEnd);
			} 
			else 
			{
				document.removeEventListener("mousemove", refJSDataGrid.dragMove.bind(refJSDataGrid), true);
				document.removeEventListener("mouseup", refJSDataGrid.dragEnd, true);
			}
		    
		  	var _dragColumns = refJSDataGrid._dragColumns;
		    if (!_dragColumns.addedNode) 
		    {
		       return;
		    }
		    _dragColumns.tableContainer.removeChild(_dragColumns.clonedTable);

		    // Determine whether the drag ended over the table, and over which column.
		    var targetPosition = getPositionFromEvent(event);
		    var tablePosition = getAbsolutePosition(_dragColumns.table);
		    if (targetPosition.y < tablePosition.y || targetPosition.y > tablePosition.y + _dragColumns.table.offsetHeight) 
		    {
		    	return;
		    }
		    var targetColumn = refJSDataGrid.getColumnByPostion(_dragColumns.table, targetPosition.x);
		    if (targetColumn != -1 && targetColumn != _dragColumns.startColumn) 
		    {
		    	refJSDataGrid.moveColumn(_dragColumns.table, _dragColumns.startColumn, targetColumn);
		    }
	  };
	  
	  this.getColumnByPostion= function(table, xPos) 
	  {
		    var header = table.tHead.rows[0].cells;
		    for (var colCount = 0; colCount < header.length; colCount++) 
		    {
		      var position = getAbsolutePosition(header[colCount]);
		      if (position.x <= xPos && xPos <= position.x + header[colCount].offsetWidth) 
		      {
		    	  return colCount;
		      }
		    }
		    return -1;
	  };
	  
	  this.copyColumnSection= function(section, columnIndex)
      {
	      var copiedSection = cloneElement(section, false);
	      forEach(section.rows, function(row) 
	      {
		        var cell = row.cells[columnIndex];
		        var newRow = cloneElement(row, false);
		        if (row.offsetHeight)
		        {
		        	newRow.style.height = row.offsetHeight + "px";
		        }
		        var newCell = cloneElement(cell, true);
		        if (cell.offsetWidth)
		        {
		        	newCell.style.width = cell.offsetWidth + "px";
		        }
		        newRow.appendChild(newCell);
		        copiedSection.appendChild(newRow);
	      });
	      return copiedSection;
      };
	  
	  // Move a column of table from start index to finish index.
	  // Assumes there are columns has to move from sourceX to destinationX
	  this.moveColumn= function(table, sourceX, destinationX) 
	  {
		    var rowLength=table.rows.length;
		    while (rowLength--)
		    {
		    	var row = table.rows[rowLength];
			    var cell = row.removeChild(row.cells[sourceX]);
			    if (destinationX < row.cells.length) 
			    {
			    	row.insertBefore(cell, row.cells[destinationX]);
			    } 
			    else 
			    {
			    	row.appendChild(cell);
			    }
		    }
	
		    // For whatever reason, sorttable tracks column indices this way.
		    // Without a manual update, clicking one column will sort on another.
		    var headrow = table.tHead.rows[0].cells;
		    for (var colCount=0; colCount<headrow.length; colCount++) 
		    {
		    	headrow[colCount].columnIndex = colCount;
		    }
	  };


	/******************************************************End of Dragging Logic*************************************************************/
}

/******************************************************Start of Util Functions*************************************************************/
function findParent(element,parentTag)
{
    if(element && parentTag)
    {
        while (element && element.tagName && element.tagName.toLowerCase()!=parentTag.toLowerCase())
        {
            element = element.parentNode;
        }
    }  
    return element;
}

function getElement(elementId)
{
    if(elementId && elementId.length > 0)
    {
        return document.getElementById(elementId);
    }
    return null;
}

function isBrowserIE()
{
    if(navigator && navigator.appName && navigator.appName == "Microsoft Internet Explorer")
    {
        return true;
    }
    return false;
}
   
function getEvent(event)
{
 // IE is evil and doesn't pass the event object
     if (!event)
     {
          event = window.event;
     }
     return event;
}
//returns target for all kind of browser
function getTarget(event)
{
     // we assume we have a standards compliant browser, but check if we have IE
     event = getEvent(event);
     var target = event.target ? event.target : event.srcElement;
     return target;
}

function addStyleClass(divAlert,styleClass)
{
    if(divAlert && styleClass && styleClass.length > 0)
    {
        if(divAlert.classList == undefined)
        {
            if(!hasStyleClass(divAlert,styleClass))
            {
                divAlert.className += " " + styleClass;
            }
        }
        else
        {
            if(!hasStyleClass(divAlert,styleClass))
            {
                divAlert.classList.add(styleClass);
            }
        }
    }
}
//returns true if a Style class is present in element's styles list
function hasStyleClass(divAlert,styleClass)
{
    if(divAlert && styleClass && styleClass.length > 0)
    {
        if(divAlert.classList == undefined)
        {
            return (divAlert.className.indexOf(" " + styleClass) > -1);
        }
        else
        {
            return (divAlert.classList.contains(styleClass));
        }
    }
    return false;
}
//removes Style class from an element's styles list
function removeStyleClass(divAlert,styleClass)
{
    if(divAlert && styleClass && styleClass.length > 0)
    {
        if(divAlert.classList == undefined)
        {
            if(divAlert.className)
            {
                divAlert.className = divAlert.className.replace(" " + styleClass,"");
            }
        }
        else
        {
            divAlert.classList.remove(styleClass);
        }
    }
}

function isFunction(refFunction)
{
    if(refFunction && typeof refFunction == "function")
    {
        return true;
    }
    return false;
}

function isObjectEqual(a, b) 
{
    // Create arrays of property names
    var aProps = Object.getOwnPropertyNames(a);
    var bProps = Object.getOwnPropertyNames(b);

    // If number of properties is different,
    // objects are not equivalent
    if (aProps.length != bProps.length) {
        return false;
    }

    for (var i = 0; i < aProps.length; i++) {
        var propName = aProps[i];

        // If values of same property are not equal,
        // objects are not equivalent
        if (a[propName] !== b[propName]) {
            return false;
        }
    }

    // If we made it this far, objects
    // are considered equivalent
    return true;
}

//change Style class from an element's styles list
function changeStyleClass(divAlert,oldStyleClass,newStyleClass)
{
    if(divAlert && oldStyleClass && oldStyleClass.length > 0 && newStyleClass && newStyleClass.length > 0)
    {
        if(divAlert.classList == undefined)
        {
            if(divAlert.className)
            {
                divAlert.className = divAlert.className.replace(oldStyleClass,newStyleClass);
            }
        }
        else
        {
            divAlert.classList.remove(oldStyleClass);
            if(!divAlert.classList.contains(newStyleClass))
            {
                divAlert.classList.add(newStyleClass);
            }
        }
    }
}

function createDiv(id,styleName)
{
    var div = document.createElement("div");  
    if(id)
    {
    	div.id = id;  
    }
    if(styleName)
    {
        div.className = styleName;   
    }
    return div;
}

function setDivVisibility(id,isVisible)
{
    var div = document.getElementById(id);
    if(div)
    {
        if(isVisible)
        {
            div.style.display = "block";
        }
        else
        {
            div.style.display = "none";
        }
    }
}

function removeDiv(id)
{
    var div = document.getElementById(id);
    if (div)
    {
        var parent = div.parentNode;
        if(parent)
        {
            parent.removeChild(div);
        }
    }
}

function getUniqueId() 
{
    var chars = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz".split("");
    var uuid = new Array(36);
    var rnd=0;
    var r;
    
    for (var count = 0; count < 36; count++) 
    {
      if (count==8 || count==13 ||  count==18 || count==23) 
      {
        uuid[count] = '-';
      } 
      else if (count==14) 
      {
        uuid[count] = '4';
      } 
      else 
      {
        if (rnd <= 0x02) 
        {
        	rnd = 0x2000000 + (Math.random()*0x1000000) | 0;
        }
        r = rnd & 0xf;
        rnd = rnd >> 4;
        uuid[count] = chars[(count == 19) ? (r & 0x3) | 0x8 : r];
      }
    }
    return uuid.join('');
}

function getPositionFromEvent(event) 
{
	var position=new Object();
	event = getEvent(event);
    if (isBrowserIE()) 
    {
    	position.x = window.event.clientX + document.documentElement.scrollLeft + document.body.scrollLeft;
    	position.y = window.event.clientY + document.documentElement.scrollTop + document.body.scrollTop;
    }
    else
    {
    	position.x = event.pageX;
    	position.y = event.pageY;
    }
    return position;
}

function getAbsolutePosition(element,doesStopAtRelativePostion) 
{
	var position=new Object();
	position.x = 0;
	position.y = 0;
    do 
    {
    	var currentStyle = isBrowserIE() ? element.currentStyle : window.getComputedStyle(element, "");
    	var supportFixed = !(isBrowserIE() && getBrowserDetails().majorVersion < 7);
    	if (doesStopAtRelativePostion && currentStyle.position == "relative") 
    	{
    		break;
    	} 
    	else if (supportFixed && currentStyle.position == "fixed") 
    	{
    		// Get the fixed elements offset
    		position.x += parseInt(currentStyle.left, 10);
    		position.y += parseInt(currentStyle.top, 10);
    		// Compensate for scrolling
    		position.x += document.body.scrollLeft;
    		position.y += document.body.scrollTop;
    		// End the loop
    		break;
    	} 
    	else 
    	{
    		position.x += element.offsetLeft;
    		position.y += element.offsetTop;
    	}
    }
    while (element = element.offsetParent);
    return position;
}

/* Ref : http://www.javascripter.net/faq/browsern.htm
   Sample Output :
   Browser name = Microsoft Internet Explorer
   Full version = 8.0
   Major version = 8
   applicationName = Microsoft Internet Explorer
   userAgent = Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; InfoPath.2; MS-RTC LM 8; .NET4.0C; .NET4.0E)
 */

function getBrowserDetails()
{
	var objBrowserDetails = new Object();
	var navigatorVersion = navigator.appVersion;
	var navigatorAgent = navigator.userAgent;
	var browserName  = navigator.appName;
	var fullVersion  = "" + parseFloat(navigator.appVersion); 
	var majorVersion = parseInt(navigator.appVersion,10);
	var nameOffset,verOffset,ix;

	// In Opera, the true version is after "Opera" or after "Version"
	if ((verOffset=navigatorAgent.indexOf("Opera"))!=-1) 
	{
		 browserName = "Opera";
		 fullVersion = navigatorAgent.substring(verOffset + 6);
		 if ((verOffset=navigatorAgent.indexOf("Version"))!=-1) 
	     {
			 fullVersion = navigatorAgent.substring(verOffset+8); 
	     }
	}
	// In MSIE, the true version is after "MSIE" in userAgent
	else if ((verOffset=navigatorAgent.indexOf("MSIE"))!=-1) 
	{
		browserName = "Microsoft Internet Explorer";
		fullVersion = navigatorAgent.substring(verOffset+5);
	}
	// In Chrome, the true version is after "Chrome" 
	else if ((verOffset=navigatorAgent.indexOf("Chrome"))!=-1) 
	{
		 browserName = "Chrome";
		 fullVersion = navigatorAgent.substring(verOffset+7);
	}
	// In Safari, the true version is after "Safari" or after "Version" 
	else if ((verOffset=navigatorAgent.indexOf("Safari"))!=-1) 
	{
		 browserName = "Safari";
		 fullVersion = navigatorAgent.substring(verOffset+7);
		 if ((verOffset=navigatorAgent.indexOf("Version"))!=-1) 
		 {
			 fullVersion = navigatorAgent.substring(verOffset+8); 
		 }
	}
	// In Firefox, the true version is after "Firefox" 
	else if ((verOffset=navigatorAgent.indexOf("Firefox"))!=-1) 
	{
		browserName = "Firefox";
		fullVersion = navigatorAgent.substring(verOffset+8);
	}
	// In most other browsers, "name/version" is at the end of userAgent 
	else if ((nameOffset=navigatorAgent.lastIndexOf(" ")+1) < (verOffset=navigatorAgent.lastIndexOf("/"))) 
	{
		 browserName = navigatorAgent.substring(nameOffset,verOffset);
		 fullVersion = navigatorAgent.substring(verOffset+1);
		 if (browserName.toLowerCase() == browserName.toUpperCase()) 
		 {
			 browserName = navigator.appName;
		 }
	}
	// trim the fullVersion string at semicolon/space if present
	if ((ix=fullVersion.indexOf(";"))!=-1)
	{
		fullVersion=fullVersion.substring(0,ix);
	}
	if ((ix=fullVersion.indexOf(" "))!=-1)
	{
		fullVersion=fullVersion.substring(0,ix);
	}
	majorVersion = parseInt("" + fullVersion,10);
	if (isNaN(majorVersion)) 
	{
		fullVersion  = "" + parseFloat(navigator.appVersion); 
		majorVersion = parseInt(navigator.appVersion,10);
	}
	
	objBrowserDetails.browserName = browserName;
	objBrowserDetails.fullVersion = fullVersion;
	objBrowserDetails.majorVersion = majorVersion;
	objBrowserDetails.applicationName = navigator.appName;
	objBrowserDetails.userAgent = navigator.userAgent;
	
	return objBrowserDetails;
}

// clone an element, copying its style and class.
function cloneElement(element,isDeepCopy) 
{
  var clonedElement = element.cloneNode(isDeepCopy);
  clonedElement.className = element.className;
  forEach(element.style, function(value, key, object) 
  {
        if (value == null) 
        {
        	return;
        }
        if (typeof(value) == "string" && value.length == 0) 
        {
        	return;
        }
        clonedElement.style[key] = element.style[key];
  });
  return clonedElement;
}

/*  Start of section copied from http://dean.edwards.name/base/forEach.js */

//array-like enumeration
if (!Array.forEach) 
{ // mozilla already supports this
  Array.forEach = function(array, block, context) 
  {
    for (var i = 0; i < array.length; i++) 
    {
      block.call(context, array[i], i, array);
    }
  };
}

// generic enumeration
Function.prototype.forEach = function(object, block, context) 
{
  for (var key in object) 
  {
    if (typeof this.prototype[key] == "undefined") 
    {
    	block.call(context, object[key], key, object);
    }
  }
};

// character enumeration
String.forEach = function(string, block, context) 
{
  Array.forEach(string.split(""), function(chr, index) 
  {
    block.call(context, chr, index, string);
  });
};

function forEach(object, block, context) 
{
	  if (object) 
	  {
		    var resolve = Object; // default
		    if (object instanceof Function) 
		    {
		      // functions have a "length" property
		      resolve = Function;
		    } 
		    else if (object.forEach instanceof Function) 
		    {
		      // the object implements a custom forEach method so use that
		      object.forEach(block, context);
		      return;
		    } 
		    else if (typeof object == "string") 
		    {
		      // the object is a string
		      resolve = String;
		    } 
		    else if (typeof object.length == "number") 
		    {
		      // the object is array-like
		      resolve = Array;
		    }
		    resolve.forEach(object, block, context);
	  }
}

function getTemplate(templateID)
{
	var compBodySpan = null;
	if(templateID)
	{
		if(supportsTemplate())
		{
			var template = document.querySelector(templateID);
			if(template)
			{
				compBodySpan = document.createElement("span");
				var templateClone = document.importNode(template.content, true);
				compBodySpan.appendChild(templateClone);
			}
		}
		else
		{
			var template = document.querySelector(templateID);
			if(template)
			{
				compBodySpan = document.createElement("span");
				compBodySpan.innerHTML = template.innerHTML;
				template.style.display = "none";
				//template.parentNode.removeChild(template);
			}
		}
	}
	
	return compBodySpan;
}

function supportsTemplate() 
{
    return ("content" in document.createElement('template'));
}

function getCumulativeOffset(element)
{
    var x = 0;
    var y = 0;
    var currentElement = (element) ? element : this;
    do
    {
        if (currentElement.nodeName.toLowerCase != 'td')
        {
            x += currentElement.offsetLeft;
            y += currentElement.offsetTop;
        }
    }
    while ((currentElement = currentElement.offsetParent) && currentElement.nodeName.toLowerCase() != 'body');   
 
    return { x: x, y: y };
}

function getScrollBarWidth(element)
{
	var scrollBarWidth = 0;
	if(element)
	{
		scrollBarWidth = element.offsetWidth - element.clientWidth;
		console.log("scrollBarWidth" + scrollBarWidth);
	}
	return scrollBarWidth;
}

/*function addEvent(element, eventType, listener)
{
	 var retValue = false;
	 if (element.addEventListener)
	 {
		 element.addEventListener(eventType, listener);
		 retValue = true;
	 } 
	 else if (element.attachEvent)
	 {
	    retValue = element.attachEvent("on" + eventType, listener);
	 } 
	 else 
	 {
	    console.log("Handler could not be attached");
	    retValue = false;
	 }
	 
	 return retValue;
}

function removeEvent(element, eventType, listener)
{
	 var retValue = false;
	 if (element.removeEventListener)
	 {
		 element.removeEventListener(eventType, listener);
		 retValue = true;
	 } 
	 else if (element.detachEvent)
	 {
	    retValue = element.detachEvent("on" + eventType, listener);
	 } 
	 else 
	 {
	    console.log("Handler could not be attached");
	    retValue = false;
	 }
	 
	 return retValue;
}*/

function getDimensionAsNumber(element,dimension)
{
	var retValue = 0;
	if(element && dimension)
	{
		if(dimension.substring(dimension.length - 1) == "%")
		{
			dimension = dimension.substring(0,dimension.length - 1);
			retValue = (dimension / 100) * element.parent.offsetHeight;
		}
		else if(dimension.substring(dimension.length - 2) == "px")
		{
			retValue = dimension.substring(0,dimension.length - 2);
		}
		else
		{
			retValue = dimension;
		}
	}
	if(isNaN(retValue))
	{
		retValue = 0;
	}
	return parseInt(retValue);
}

function makeUnselectable(node) 
{
    if (node.nodeType == 1) 
    {
        node.setAttribute("unselectable", "on");
    }
    var child = node.firstChild;
    while (child) 
    {
        makeUnselectable(child);
        child = child.nextSibling;
    }
}

function getFirstChild(element)
{
	var firstChild = null;
	if(element)
	{
		firstChild = element.firstChild;
		while (firstChild.nodeType != 1)
		{
			firstChild = firstChild.nextSibling;
		}
	}
	return firstChild;
}

/*  End of section copied from http://dean.edwards.name/base/forEach.js */

if (!Function.prototype.bind) 
{
	  Function.prototype.bind = function (oThis) 
	  {
	    if (typeof this !== "function") 
	    {
	      // closest thing possible to the ECMAScript 5 internal IsCallable function
	      throw new TypeError("Function.prototype.bind - what is trying to be bound is not callable");
	    }

	    var aArgs = Array.prototype.slice.call(arguments, 1),
	        fToBind = this,
	        fNOP = function () {},
	        fBound = function () 
	        {
	          return fToBind.apply(this instanceof fNOP && oThis
	                                 ? this
	                                 : oThis,
	                               aArgs.concat(Array.prototype.slice.call(arguments)));
	        };

	    fNOP.prototype = this.prototype;
	    fBound.prototype = new fNOP();

	    return fBound;
	  };
}

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<!-- css from http://www.csstablegenerator.com/?table_id=3 -->
<style>

.dataGrid
{
    /*border-collapse: collapse;
    border-spacing: 0;
    margin:0px;
    padding:0px;
    border:1px solid #000000;*/
    
    width: 100%;
background-color: #EEE;
border: 0px;
margin: 0px;
padding: 0px;
background-color: #F6F6F6;
border-collapse: collapse;
}
.dataGridTitleBar
{
    background-color: #eee;
    border-left: 1px solid #ccc;
    border-right: 1px solid #ccc;
    border-top: 1px solid #ccc;
    color: #333;
    font-weight: bold;
    text-shadow: 1px 1px 0 white;
}

.dataGridHeader
{
    /*background:-o-linear-gradient(bottom, #005fbf 5%, #003f7f 100%);  
    background:-webkit-gradient( linear, left top, left bottom, color-stop(0.05, #005fbf), color-stop(1, #003f7f) );
    background:-moz-linear-gradient( center top, #005fbf 5%, #003f7f 100% );
    filter:progid:DXImageTransform.Microsoft.gradient(startColorstr="#005fbf", endColorstr="#003f7f");  
    background: -o-linear-gradient(top,#005fbf,003f7f);
    background-color:#005fbf;
    border:1px solid #000000;
    text-align:center;
    border-width:0px 0px 1px 1px;
    font-weight:bold;
    color:#ffffff;
    cursor : default;*/
    background-color: #888888;
  	color: White;
  	line-height: 1.8em;
  	margin: 0px;
  white-space: nowrap;
   border-left: 1px solid #000000;
  padding: 4px 6px 2px;
    border-collapse: collapse;
}
.dataGridOddRow
{
    background-color: white;
	color: black;
}
.dataGridEvenRow  
{
    background-color: #E0E0E0;
	color: black; 
}
.dataGridCell
{
    border-bottom : buttonshadow 1px solid;
    border-top : buttonshadow 1px solid;
    border-left : buttonshadow 1px solid;
    border-right : buttonshadow 1px solid;
    cursor : default;
    padding:7px;
    text-align:left;
    font-weight:normal;
     vertical-align:middle;
     color:#000000;
}
.dataGridHover
{
    background-color: #CCCCCC;
}
.dataGridSelection
{
    background-color: #b0bed9;
}

.hbox {
  overflow-x:auto;
}
.hbox > * {
  display: inline-block;
  padding: 3px;
}


/*.hbox:before,.hbox:after
{
	display: table;
  	content: " ";
}

.hbox:after
{
	clear: both;
}
*/

.arrow-down 
{
	/*width: 0; 
	height: 0; 
	border-left: 5px solid transparent;
	border-right: 5px solid transparent;
	border-top: 5px solid #000;*/
	
	width: 0;
	height: 0;
	border-style: solid;
	border-width: 10px 5px 0 5px;
	border-color: #1f1c1c transparent transparent transparent;
}

.arrow-right 
{
	/*width: 0; 
	height: 0; 
	border-top: 5px solid transparent;
	border-bottom: 5px solid transparent;
	border-left: 5px solid #000;*/
	
	width: 0;
	height: 0;
	border-style: solid;
	border-width: 4px 0 4px 10px;
	border-color: transparent transparent transparent #1f1c1c;
}

.resize-handle-active
{
	cursor: e-resize;
}

.resize-handle 
{
	cursor: e-resize;
	width: 2px;
	border-right: 1px dashed #1E90FF;
	position:absolute;
	top:0;
	left:0;
}

*.unselectable {
   -moz-user-select: none;
   -khtml-user-select: none;
   -webkit-user-select: none;

   /*
     Introduced in IE 10.
     See http://ie.microsoft.com/testdrive/HTML5/msUserSelect/
   */
   -ms-user-select: none;
   user-select: none;
}

</style>

<script type="text/javascript" src="jsDataGrid1.js"></script>
<!--  <script type="text/javascript" src="resizable-tables.js"></script> -->
<script type="text/javascript">

var columns=[ { headerText: "Id", dataField: "id", width : 80 , sortable: true , sortDescending: false },
              { headerText: "Hierarchy", dataField: "hierarchy" ,width : 200 , sortable: true , sortDescending: true },
              { headerText: "Supervisor", dataField: "supervisor" ,width : 150 , sortable: false , sortDescending: true },
              { headerText: "Employees", dataField: "employees" ,width : 150 , sortable: true , sortDescending: true },
              //{ headerText: "", dataField: "checked" ,width : 50 , sortable: false , sortDescending: false,templateID:"#tempCheckBox"},
              { headerText: "", dataField: "checked" ,width : 50 , sortable: false , sortDescending: false,itemRenderer:itemRenderer}
           ];

                
var dataSource =[{id: '1', hierarchy: 'NDPI', supervisor: null, country: 'US', employees: null, price: '10.90', year: '1985',checked:true,children:[
					{id: '2', hierarchy: 'NGFP Head', supervisor: 'Bonnie Tyler', country: 'UK', employees: 'Patel,Samir', price: '9.90', year: '1988',children:[
						{id: '3', hierarchy: 'NGFP Corporate Equity Derivative Sales', supervisor: 'Bee Gees', country: 'UK', employees: 'Polydor', price: '10.90', year: '1998'},
						{id: '4', hierarchy: 'NGFP Structured Products Sales', supervisor: 'Andrea Bocelli', country: 'US', employees: 'Polydor', price: '10.80', year: '1996'}  					                                                                                                                                         
					]},
                 ]},
	             {id: '5', hierarchy: 'Non Regulated Entity', supervisor: null, country: 'US', employees: null, price: '10.90', year: '1985',children:[
					{id: '6', hierarchy: 'Nomura America Services, LLC', supervisor: 'Bonnie Tyler', country: 'UK', employees: 'Patel,Samir', price: '9.90', year: '1988',children:[
						{id: '7', hierarchy: 'OPERATIONS', supervisor: 'Bee Gees', country: 'UK', employees: 'Polydor', price: '10.90', year: '1998'},
						{id: '8', hierarchy: 'ENTERPRISE DATA MGMT', supervisor: 'Andrea Bocelli', country: 'US', employees: 'Polydor', price: '10.80', year: '1996'}  					                                                                                                                                         
					]},
                ]},
	            {id: '9', hierarchy: 'For the good times', supervisor: 'Kenny Rogers', country: 'UK', employees: 'Mucik Master', price: '8.70', year: '1995'},
	            {id: '10',hierarchy: 'Big Willie style', supervisor: 'Will Smith', country: 'US', employees: 'Columbia', price: '9.90', year: '1997'},
	            {id: '11', hierarchy: 'Empire Burlesque', supervisor: 'Bob Dylan', country: 'US', employees: 'Columbia', price: '10.90', year: '1985'},
	            {id: '12', hierarchy: 'Hide your heart', supervisor: 'Bonnie Tyler', country: 'UK', employees: 'CBS Records', price: '9.90', year: '1988'},
	            {id: '13', hierarchy: 'One night only', supervisor: 'Bee Gees', country: 'UK', employees: 'Polydor', price: '10.90', year: '1998'},
	            {id: '14', hierarchy: 'Romanza', supervisor: 'Andrea Bocelli', country: 'US', employees: 'Polydor', price: '10.80', year: '1996'},
	            {id: '15', hierarchy: 'Pavarotti Gala Concert', supervisor: 'Luciano Pavarotti', country: 'US', employees: 'DECCA', price: '9.90', year: '1991'},
	            {id: '16', hierarchy: 'Picture book', supervisor: 'Simply Red', country: 'US', employees: 'Elektra', price: '7.90', year: '1985'},
	            {id: '17', hierarchy: 'Eros', supervisor: 'Eros Ramazzotti', country: 'US', employees: 'BMG', price: '9.90', year: '1997'},
	            {id: '18', hierarchy: 'Black angel', supervisor: 'Savage Rose', country: 'US', employees: 'Mega', price: '10.90', year: '1995'},
	            {id: '19', hierarchy: 'For the good times', supervisor: 'Kenny Rogers', country: 'UK', employees: 'Mucik Master', price: '8.70', year: '1995'},
	            {id: '20',hierarchy: 'Big Willie style', supervisor: 'Will Smith', country: 'US', employees: 'Columbia', price: '9.90', year: '1997'}
	            /* {id: '21', hierarchy: 'Empire Burlesque', supervisor: 'Bob Dylan', country: 'US', employees: 'Columbia', price: '10.90', year: '1985'},
	            {id: '22', hierarchy: 'Hide your heart', supervisor: 'Bonnie Tyler', country: 'UK', employees: 'CBS Records', price: '9.90', year: '1988'},
	            {id: '23', hierarchy: 'One night only', supervisor: 'Bee Gees', country: 'UK', employees: 'Polydor', price: '10.90', year: '1998'},
	            {id: '24', hierarchy: 'Romanza', supervisor: 'Andrea Bocelli', country: 'US', employees: 'Polydor', price: '10.80', year: '1996'},
	            {id: '25', hierarchy: 'Pavarotti Gala Concert', supervisor: 'Luciano Pavarotti', country: 'US', employees: 'DECCA', price: '9.90', year: '1991'},
	            {id: '26', hierarchy: 'Picture book', supervisor: 'Simply Red', country: 'US', employees: 'Elektra', price: '7.90', year: '1985'},
	            {id: '27', hierarchy: 'Eros', supervisor: 'Eros Ramazzotti', country: 'US', employees: 'BMG', price: '9.90', year: '1997'},
	            {id: '28', hierarchy: 'Black angel', supervisor: 'Savage Rose', country: 'US', employees: 'Mega', price: '10.90', year: '1995'},
	            {id: '29', hierarchy: 'For the good times', supervisor: 'Kenny Rogers', country: 'UK', employees: 'Mucik Master', price: '8.70', year: '1995'},
	            {id: '30',hierarchy: 'Big Willie style', supervisor: 'Will Smith', country: 'US', employees: 'Columbia', price: '9.90', year: '1997'},
	            {id: '31', hierarchy: 'Empire Burlesque', supervisor: 'Bob Dylan', country: 'US', employees: 'Columbia', price: '10.90', year: '1985'},
	            {id: '32', hierarchy: 'Hide your heart', supervisor: 'Bonnie Tyler', country: 'UK', employees: 'CBS Records', price: '9.90', year: '1988'},
	            {id: '33', hierarchy: 'One night only', supervisor: 'Bee Gees', country: 'UK', employees: 'Polydor', price: '10.90', year: '1998'},
	            {id: '34', hierarchy: 'Romanza', supervisor: 'Andrea Bocelli', country: 'US', employees: 'Polydor', price: '10.80', year: '1996'},
	            {id: '35', hierarchy: 'Pavarotti Gala Concert', supervisor: 'Luciano Pavarotti', country: 'US', employees: 'DECCA', price: '9.90', year: '1991'},
	            {id: '36', hierarchy: 'Picture book', supervisor: 'Simply Red', country: 'US', employees: 'Elektra', price: '7.90', year: '1985'},
	            {id: '37', hierarchy: 'Eros', supervisor: 'Eros Ramazzotti', country: 'US', employees: 'BMG', price: '9.90', year: '1997'},
	            {id: '38', hierarchy: 'Black angel', supervisor: 'Savage Rose', country: 'US', employees: 'Mega', price: '10.90', year: '1995'},
	            {id: '39', hierarchy: 'For the good times', supervisor: 'Kenny Rogers', country: 'UK', employees: 'Mucik Master', price: '8.70', year: '1995'},
	            {id: '40',hierarchy: 'Big Willie style', supervisor: 'Will Smith', country: 'US', employees: 'Columbia', price: '9.90', year: '1997'},
	            {id: '41', hierarchy: 'Empire Burlesque', supervisor: 'Bob Dylan', country: 'US', employees: 'Columbia', price: '10.90', year: '1985'},
	            {id: '42', hierarchy: 'Hide your heart', supervisor: 'Bonnie Tyler', country: 'UK', employees: 'CBS Records', price: '9.90', year: '1988'},
	            {id: '43', hierarchy: 'One night only', supervisor: 'Bee Gees', country: 'UK', employees: 'Polydor', price: '10.90', year: '1998'},
	            {id: '44', hierarchy: 'Romanza', supervisor: 'Andrea Bocelli', country: 'US', employees: 'Polydor', price: '10.80', year: '1996'},
	            {id: '45', hierarchy: 'Pavarotti Gala Concert', supervisor: 'Luciano Pavarotti', country: 'US', employees: 'DECCA', price: '9.90', year: '1991'},
	            {id: '46', hierarchy: 'Picture book', supervisor: 'Simply Red', country: 'US', employees: 'Elektra', price: '7.90', year: '1985'},
	            {id: '47', hierarchy: 'Eros', supervisor: 'Eros Ramazzotti', country: 'US', employees: 'BMG', price: '9.90', year: '1997'},
	            {id: '48', hierarchy: 'Black angel', supervisor: 'Savage Rose', country: 'US', employees: 'Mega', price: '10.90', year: '1995'},
	            {id: '49', hierarchy: 'For the good times', supervisor: 'Kenny Rogers', country: 'UK', employees: 'Mucik Master', price: '8.70', year: '1995'},
	            {id: '50',hierarchy: 'Big Willie style', supervisor: 'Will Smith', country: 'US', employees: 'Columbia', price: '9.90', year: '1997'},
	            {id: '51', hierarchy: 'Empire Burlesque', supervisor: 'Bob Dylan', country: 'US', employees: 'Columbia', price: '10.90', year: '1985'},
	            {id: '52', hierarchy: 'Hide your heart', supervisor: 'Bonnie Tyler', country: 'UK', employees: 'CBS Records', price: '9.90', year: '1988'},
	            {id: '53', hierarchy: 'One night only', supervisor: 'Bee Gees', country: 'UK', employees: 'Polydor', price: '10.90', year: '1998'},
	            {id: '54', hierarchy: 'Romanza', supervisor: 'Andrea Bocelli', country: 'US', employees: 'Polydor', price: '10.80', year: '1996'},
	            {id: '55', hierarchy: 'Pavarotti Gala Concert', supervisor: 'Luciano Pavarotti', country: 'US', employees: 'DECCA', price: '9.90', year: '1991'},
	            {id: '56', hierarchy: 'Picture book', supervisor: 'Simply Red', country: 'US', employees: 'Elektra', price: '7.90', year: '1985'},
	            {id: '57', hierarchy: 'Eros', supervisor: 'Eros Ramazzotti', country: 'US', employees: 'BMG', price: '9.90', year: '1997'},
	            {id: '58', hierarchy: 'Black angel', supervisor: 'Savage Rose', country: 'US', employees: 'Mega', price: '10.90', year: '1995'},
	            {id: '59', hierarchy: 'For the good times', supervisor: 'Kenny Rogers', country: 'UK', employees: 'Mucik Master', price: '8.70', year: '1995'},
	            {id: '60',hierarchy: 'Big Willie style', supervisor: 'Will Smith', country: 'US', employees: 'Columbia', price: '9.90', year: '1997'},
	            {id: '61', hierarchy: 'Empire Burlesque', supervisor: 'Bob Dylan', country: 'US', employees: 'Columbia', price: '10.90', year: '1985'},
	            {id: '62', hierarchy: 'Hide your heart', supervisor: 'Bonnie Tyler', country: 'UK', employees: 'CBS Records', price: '9.90', year: '1988'},
	            {id: '63', hierarchy: 'One night only', supervisor: 'Bee Gees', country: 'UK', employees: 'Polydor', price: '10.90', year: '1998'},
	            {id: '64', hierarchy: 'Romanza', supervisor: 'Andrea Bocelli', country: 'US', employees: 'Polydor', price: '10.80', year: '1996'},
	            {id: '65', hierarchy: 'Pavarotti Gala Concert', supervisor: 'Luciano Pavarotti', country: 'US', employees: 'DECCA', price: '9.90', year: '1991'},
	            {id: '66', hierarchy: 'Picture book', supervisor: 'Simply Red', country: 'US', employees: 'Elektra', price: '7.90', year: '1985'},
	            {id: '67', hierarchy: 'Eros', supervisor: 'Eros Ramazzotti', country: 'US', employees: 'BMG', price: '9.90', year: '1997'},
	            {id: '68', hierarchy: 'Black angel', supervisor: 'Savage Rose', country: 'US', employees: 'Mega', price: '10.90', year: '1995'},
	            {id: '69', hierarchy: 'For the good times', supervisor: 'Kenny Rogers', country: 'UK', employees: 'Mucik Master', price: '8.70', year: '1995'},
	            {id: '70',hierarchy: 'Big Willie style', supervisor: 'Will Smith', country: 'US', employees: 'Columbia', price: '9.90', year: '1997'},
	            {id: '71', hierarchy: 'Empire Burlesque', supervisor: 'Bob Dylan', country: 'US', employees: 'Columbia', price: '10.90', year: '1985'},
	            {id: '72', hierarchy: 'Hide your heart', supervisor: 'Bonnie Tyler', country: 'UK', employees: 'CBS Records', price: '9.90', year: '1988'},
	            {id: '73', hierarchy: 'One night only', supervisor: 'Bee Gees', country: 'UK', employees: 'Polydor', price: '10.90', year: '1998'},
	            {id: '74', hierarchy: 'Romanza', supervisor: 'Andrea Bocelli', country: 'US', employees: 'Polydor', price: '10.80', year: '1996'},
	            {id: '75', hierarchy: 'Pavarotti Gala Concert', supervisor: 'Luciano Pavarotti', country: 'US', employees: 'DECCA', price: '9.90', year: '1991'},
	            {id: '76', hierarchy: 'Picture book', supervisor: 'Simply Red', country: 'US', employees: 'Elektra', price: '7.90', year: '1985'},
	            {id: '77', hierarchy: 'Eros', supervisor: 'Eros Ramazzotti', country: 'US', employees: 'BMG', price: '9.90', year: '1997'},
	            {id: '78', hierarchy: 'Black angel', supervisor: 'Savage Rose', country: 'US', employees: 'Mega', price: '10.90', year: '1995'},
	            {id: '79', hierarchy: 'For the good times', supervisor: 'Kenny Rogers', country: 'UK', employees: 'Mucik Master', price: '8.70', year: '1995'},
	            {id: '80',hierarchy: 'Big Willie style', supervisor: 'Will Smith', country: 'US', employees: 'Columbia', price: '9.90', year: '1997'},
	            {id: '81', hierarchy: 'Empire Burlesque', supervisor: 'Bob Dylan', country: 'US', employees: 'Columbia', price: '10.90', year: '1985'},
	            {id: '82', hierarchy: 'Hide your heart', supervisor: 'Bonnie Tyler', country: 'UK', employees: 'CBS Records', price: '9.90', year: '1988'},
	            {id: '83', hierarchy: 'One night only', supervisor: 'Bee Gees', country: 'UK', employees: 'Polydor', price: '10.90', year: '1998'},
	            {id: '84', hierarchy: 'Romanza', supervisor: 'Andrea Bocelli', country: 'US', employees: 'Polydor', price: '10.80', year: '1996'},
	            {id: '85', hierarchy: 'Pavarotti Gala Concert', supervisor: 'Luciano Pavarotti', country: 'US', employees: 'DECCA', price: '9.90', year: '1991'},
	            {id: '86', hierarchy: 'Picture book', supervisor: 'Simply Red', country: 'US', employees: 'Elektra', price: '7.90', year: '1985'},
	            {id: '87', hierarchy: 'Eros', supervisor: 'Eros Ramazzotti', country: 'US', employees: 'BMG', price: '9.90', year: '1997'},
	            {id: '88', hierarchy: 'Black angel', supervisor: 'Savage Rose', country: 'US', employees: 'Mega', price: '10.90', year: '1995'},
	            {id: '89', hierarchy: 'For the good times', supervisor: 'Kenny Rogers', country: 'UK', employees: 'Mucik Master', price: '8.70', year: '1995'},
	            {id: '90',hierarchy: 'Big Willie style', supervisor: 'Will Smith', country: 'US', employees: 'Columbia', price: '9.90', year: '1997'},
	            {id: '91', hierarchy: 'Empire Burlesque', supervisor: 'Bob Dylan', country: 'US', employees: 'Columbia', price: '10.90', year: '1985'},
	            {id: '92', hierarchy: 'Hide your heart', supervisor: 'Bonnie Tyler', country: 'UK', employees: 'CBS Records', price: '9.90', year: '1988'},
	            {id: '93', hierarchy: 'One night only', supervisor: 'Bee Gees', country: 'UK', employees: 'Polydor', price: '10.90', year: '1998'},
	            {id: '94', hierarchy: 'Romanza', supervisor: 'Andrea Bocelli', country: 'US', employees: 'Polydor', price: '10.80', year: '1996'},
	            {id: '95', hierarchy: 'Pavarotti Gala Concert', supervisor: 'Luciano Pavarotti', country: 'US', employees: 'DECCA', price: '9.90', year: '1991'},
	            {id: '96', hierarchy: 'Picture book', supervisor: 'Simply Red', country: 'US', employees: 'Elektra', price: '7.90', year: '1985'},
	            {id: '97', hierarchy: 'Eros', supervisor: 'Eros Ramazzotti', country: 'US', employees: 'BMG', price: '9.90', year: '1997'},
	            {id: '98', hierarchy: 'Black angel', supervisor: 'Savage Rose', country: 'US', employees: 'Mega', price: '10.90', year: '1995'},
	            {id: '99', hierarchy: 'For the good times', supervisor: 'Kenny Rogers', country: 'UK', employees: 'Mucik Master', price: '8.70', year: '1995'},
	            {id: '100', hierarchy: 'Big Willie style', supervisor: 'Will Smith', country: 'US', employees: 'Columbia', price: '9.90', year: '1997'}*/
            ];
var dataGrid = null;
function loadHandler()
{
	/*JSDataGrid.init("divTable",columns,"Data Set Demo",true,true);
	JSDataGrid.dataSource(dataSource);
	JSDataGrid.rowSelectionHandler = onSelection;
	JSDataGrid.rowUnSelectionHandler = onUnSelection;*/
	dataGrid = new JSDataGrid("divTable",null,"700px","700px",columns,"Data Set Demo",true,true);
	dataGrid.init();
	dataGrid.dataSource(dataSource);
	dataGrid.rowSelectionHandler = onSelection;
	dataGrid.rowUnSelectionHandler = onUnSelection;
}

function onSelection(item)
{
	if(item)
	{
		//divSelected.innerHTML += "Engine:" + item.engine + ",";
	}
}

function onUnSelection(item)
{
	if(item)
	{
		//divUnSelected.innerHTML += "Engine:" + item.engine + ",";
	}
}

function itemRenderer(data,dataField,rowIndex,columnIndex)
{
	var selected = data[dataField];
	if(selected)
	{
		return '<input type="checkbox" checked>';
	}
	else
	{
		return '<input type="checkbox">';
	}
}
 
//Global Exception Handler
/*window.onerror = function(msg, url, line)
{
    // You can view the information in an alert to see things working
    // like so:
   alert("Error: " + msg + "\nurl: " + url + "\nline #: " + line);
   // TODO: Report this error via ajax so you can keep track
   //       of what pages have JS issues
   var suppressErrorAlert = true;
   // If you return true, then error alerts (like in older versions of
   // Internet Explorer) will be suppressed.
   return suppressErrorAlert;
};*/
</script>
</head>
<body onload="loadHandler();" style="margin:0px;padding:0px;width:1000px;height:1000px;">
 <template id="tempCheckBox">
 	<input type="checkbox" name="vehicle" value="Bike">
 </template>
 <div id="divTable" style="position:relative; width: 640px; height: 700px;padding-left:50px;">
  <div class="vertical-line"></div>
 </div>
 <div id="divSelected">
 </div>
  <div id="divUnSelected">
 </div>
 <br/><br/>
<form>
  Enter column no: <input type='text' name=col_no>
  <input type='button' onClick='javascript:dataGrid.setVisibilityOfColumn(col_no.value,  true);' value='show'>
  <input type='button' onClick='javascript:dataGrid.setVisibilityOfColumn(col_no.value, false);' value='hide'>

</form>
 
 
</body>
</html>
