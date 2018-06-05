FOR PRIVACY AND CODE PROTECTING REASONS THIS IS A SIMPLIFIED VERSION OF CHANGES AND NEW FEATURES

TASK DATE: 29.05.2018 - FINISHED: 05.06.2018

TASK LEVEL: MEDIUM

TASK SHORT DESCRIPTION: [Add ordering/reordering system/arrows to the users groups in Records > User Groups. (We have an 'ordering_count' field in default_person_type)]

GITHUB REPOSITORY CODE: feature/task-1575-ordering-records-in-groups

HELPER REPOSITORY: feature/task-1084-list-ordering-for-shop

HELPER URLS: /admin-portal/content/shop/products
			 /admin-portal/members/user-groups
	
CHANGES
  
	IN FILES: 
	
		css\move_table_rows.css
	
			ADDED/CHANGED CODE
			
				.shop-product,
				tr.user-group {
					background-color: white;
					transition: background-color 1s;
				}
				.shop-product.flash-background, 
				tr.person-type.flash-background {
					background-color: #a7e886;
					transition: background-color 1s;
				}

				tr.person-type td.arrows {
					text-align: right;
				}
	
	
	
		models\profile_m.php
	
			ADDED NEW FUNCTIONS 

				/* change order of person types in person_type table 
				 * @input
				 *		- $id : int : id of person type which will be moved
				 *		- $direction: string : can be up or down
				 * @output
				 *		- boolean : true or false result of db action
				 * 
				*/
				public function setPersonTypesOrder($id, $direction)
				{
					$table = $this->db->dbprefix('person_type');
					
					$personTypeOrder = $this->getPersonTypeFieldValue($id, 'ordering_count');
									
					//If direction is up, then the person type's order will be decreased by one, so we need to replace the record with previous one
					//If direction is down, then the person type's order will be increased by one, so we need to replace the record with next one
					$personType = ($direction == 'up') 
									? $this->getPersonTypeByOrder($personTypeOrder, 'prev')
									: $this->getPersonTypeByOrder($personTypeOrder, 'next');

					if ( count($personType) > 0 ) { 				
						............
						return true;
					}
					else {
						return false;
					}
				}//END function setPersonTypesOrder
				
				
				/* getting previous or next person type record by ordering_count
				 * @input
				 *		- $personTypeOrder : int : ordering_count value in 
				 *		- $position: string : can be prev or next (if empty, in that case result is an empty array)
				 * @output
				 *		- boolean : array : record from person_type table, or empty array
				 * 
				*/
				public function getPersonTypeByOrder($personTypeOrder, $position) 
				{
					$table = $this->db->dbprefix('person_type');
					
					if ( $position == 'next' ) {
						.............
						
						return $this->db->get()->result_array();
					}
					else if ( $position == 'prev' ) {
						.............

						return $this->db->get()->result_array();
					}
					else {
						return array();
					}
				}//END function getPersonTypeByOrder			
	
				
				public function getPersonTypeFieldValue($id, $field = '')
				{
					$field = ($field != '') ? $field : 'person_type_name';
					
					.............
							
					return $res[0]->$field;
				}//END function getPersonTypeFieldValue	
				
				
	
	
		controllers\members.php
		
			ADDED NEW FUNCTION:
			
				 /* Set order of shop products */
				public function ajaxSetPersonTypeOrder()
				{ 
					if ( $this->input->is_ajax_request() ) {
						echo $this->profile_m->setPersonTypesOrder($this->input->post('id'), $this->input->post('direction')) ? true : false;			
					}
					
					echo false;
					
					exit();
				}//END function ajaxSetPersonTypeOrder
				
			CHANGED CODE 
				
				Inside function user_groups
				
					.....
					->append_css('module::user_groups.css')
					->append_js('module::user_groups.js')
					.....
					
					
	
		css\user_groups.css
		
			ADDED CODE: 
			
				tr.person-type td {
					border-top: solid 1px #ffffff !important;
					border-bottom: solid 1px #dddddd;
				}
					
		
		
		
		partials\user_groups.php

			CHANGED CODE: 
			
				FROM:
					<? foreach($person_type_options as $person_type): ?>
						<div class="accordion-heading">
							.............
						</div>
						<div id="collapse-<?=$person_type['id']?>" data-group-id="<?=$person_type['id']?>" class="accordion-body collapse">
							<div class="accordion-inner" id="questions-container-<?=$person_type['id']?>">
								<p>Loading questions...</p>
							</div>
						</div>
					<? endforeach; ?>
					<script>
					$(document).ready(function(){
						$('[data-toggle="tooltip"]').tooltip();
					});
					</script>
			
				TO: 
					<table class="table table-condensed table-list table-person-types">
						<tbody>
							<? 
								$titleUp = lang('network_settings:usergroups:title:moving_group_up_in_order'); 
								$titleDown = lang('network_settings:usergroups:title:moving_group_down_in_order'); 	
								$counter = 1; 
								$l = count($person_type_options);		
								foreach($person_type_options as $personType): 
							?>
							<tr class="person-type" data-row-counter="<?=$counter?>" id="person_type_<?=$personType['id']?>" data-post-id="<?=$personType['id']?>">
								<td>
									<div>
										.............
									</div>
									<div id="collapse-<?=$personType['id']?>" data-group-id="<?=$personType['id']?>" class="accordion-body collapse">
										<div class="accordion-inner" id="questions-container-<?=$personType['id']?>">
											<p>Loading questions...</p>
										</div>
									</div>
								</td>
								<td class="arrows">
									<?php 
										$print = ($l == 1) 
													? 	'<span class="person-type glyphicon glyphicon-chevron-up inactive"></span><br>' . 
														'<span class="person-type glyphicon glyphicon-chevron-down inactive"></span>'
													:	'';
										$print = ($counter == 1) 
													? 	'<span class="person-type glyphicon glyphicon-chevron-up inactive"></span><br>' . 
														'<span class="person-type glyphicon glyphicon-chevron-down active" title="' . $titleDown . '"></span>'
													:	$print;
										$print = ($counter == $l)
													?	'<span class="person-type glyphicon glyphicon-chevron-up active" title="' . $titleUp . '"></span><br>' . 
														'<span class="person-type glyphicon glyphicon-chevron-down inactive"></span><br>'
													:	$print; 
										echo ($print == '') 
													?	'<span class="person-type glyphicon glyphicon-chevron-up active" title="' . $titleUp . '"></span><br>' . 
														'<span class="person-type glyphicon glyphicon-chevron-down active" title="' . $titleDown . '"></span>' 
													:	$print;  
										$counter++; 
									?>			
								</td>
							</tr>
							<? endforeach; ?>
						<tbody>
					</table>


					<script>
						$(document).ready(function(){
							$('[data-toggle="tooltip"]').tooltip();
						});
					</script>				
				
		
		
		js\move_table_rows.js
		
			ALL CONTENT CHANGED
			
				/*
				 * 	Moving table rows up and down using glyphicon arrows
				 *
				 *	Author: Lajos Deli alias Lali - lajos@toucantech.com
				 *
				 * 	Package:
				 *		.............
				 *
				 *	Structure of MTR = $.movingTableRows
				 *
				 *	Callable variables
				 *
				 *	MTR.row_animation	- css class which will be set after the table rows were replaced - note - table row can not be animated, if you'd like then need to be wrapped into div		 
				 *	MTR.set_timeout 	- css class will be deleted after this time
				 *	
				 *	
				 *	Callable functions
				 *	
				 * 	MTR.setOrder(listName, arrowElement, direction, ajaxRequestUrl)
				 * 						@param listName: name of the list, for example: shop-product, generally css selector of tr element
				 * 						@param arrowElement: dom object, generally the up and down arrows 
				 * 						@param direction: can be up or down 
				 * 						@param ajaxRequestUrl: path to the ajax function 
				 * 						@param trIdPart: part of table's tr id can be: product, user_group, ....etc, if empty string, then default is product
				 * 						@param consoleType:	if you'd like to console/alert the response : can be: console, alert or empty
				 *	
				*/
					
				
				
				(function($) {
						
					$.movingTableRows = {
							
						//Variables
						row_animation 		: "flash-background",
						set_timeout 		: 500,
						
						//Methods
						setOrder : function(listName, arrowElement, direction, ajaxRequestUrl, trIdPart, consoleType) 
						{
							//Setting order on table rows if table rows with "listName" css classnames exist
							if ( $('.' + listName)[0] ) {
								.............
								
								return true;
							}
						},	

						
						
						_moveUp : function(listName, arrowElement, rowCounter) 
						{
							var row = arrowElement.parent().parents("tr:first");			
							var rows = arrowElement.parent().parent().parent().find("tr." + listName);	
							row.insertBefore(row.prev());
							MTR._setNewOrder(listName, rowCounter, 'up');
							MTR._animationAfterMoving(row, MTR.row_animation, MTR.set_timeout);
							
							if (rowCounter == rows.length) {
								.............
							}
							else if (rowCounter == 2) {
								.............
							}
						},
						

						
						_moveDown : function(listName, arrowElement, rowCounter) 
						{
							var row = arrowElement.parent().parents("tr:first");
							var rows = arrowElement.parent().parent().parent().find("tr." + listName);			
							row.insertAfter(row.next());
							MTR._setNewOrder(listName, rowCounter, 'down');
							MTR._animationAfterMoving(row, MTR.row_animation, MTR.set_timeout);
							
							if (rowCounter == 1) {
								.............
							}
							else if (rowCounter == rows.length - 1) {
								.............
							}
						},
						
						
						
						
						_setNewOrder : function(listName, rowCounter, direction) 
						{
							var new_rowCounter = (direction == 'up') ? parseInt(rowCounter) - 1 : parseInt(rowCounter) + 1;
							var element1 = $('.' + listName + '[data-row-counter="' + rowCounter + '"]');
							var element2 = $('.' + listName + '[data-row-counter="' + new_rowCounter + '"]');
							
							element1.attr('data-row-counter', new_rowCounter);
							element2.attr('data-row-counter', rowCounter);
						},

						
						
						_animationAfterMoving : function(dom_element, animation_css_class, speed) 
						{
							dom_element.addClass(animation_css_class);
							setTimeout(function() {
								dom_element.removeClass(animation_css_class);
							}, speed);
						},	
						
						
						
						_setArrowsClass : function(listName, rowCounter, up_addclass, down_addclass)		
						{
							var arrow_up = $('.' + listName + '[data-row-counter="' + rowCounter + '"]').find('span.glyphicon-chevron-up');
							var arrow_down = $('.' + listName + '[data-row-counter="' + rowCounter + '"]').find('span.glyphicon-chevron-down');

							arrow_up.removeClass('inactive').removeClass('active').addClass(up_addclass);
							arrow_down.removeClass('inactive').removeClass('active').addClass(down_addclass);
						},					
						
					}; //END $.movingTableRows object

					
					//GLOBAL variables of $.movingTableRows object
					MTR	= $.movingTableRows;
					
				})(jQuery);
				
				
				
				//Things TO DO When DOM is READY (not fully loaded)
				$(function(){
					//Reorder shop product list clicking on up/down arrows here: admin-portal/content/shop/products
					//MTR = short form of moveTableRows see above 
					if ( $('.shop-product.glyphicon-chevron-up.active')[0] ) {
						.............
					}
					if ( $('.shop-product.glyphicon-chevron-down.active')[0] ) {
						.............			
					}

					//Reorder person types list clicking on up/down arrows, link here: admin-portal/members/user-groups
					//MTR = short form of moveTableRows see above
					if ( $('.person-type.glyphicon-chevron-up.active')[0] ) {
						.............
					}
					if ( $('.person-type.glyphicon-chevron-down.active')[0] ) {
						.............		
					}	
				})
