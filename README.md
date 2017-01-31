<?php
if(!defined('sugarEntry') || !sugarEntry) die('Not A Valid Entry Point');

class custom_doc_automation_revision
{

	 function update_doc_revision(&$bean, $event, $arguments){
	 
	 
		global $db;
		$task_automation_array = array('INITIAL_PROPOSAL','DETAILED_DESIGN');
		$now = new DateTime(NULL, new DateTimeZone('Asia/Kolkata'));


		if($bean->revision != '1'){
			
		
			$original_document_id = $bean->document_id;
			$original_doc_bean = BeanFactory::getBean('Documents',$original_document_id);
			if(in_array($original_doc_bean->category_id,$task_automation_array)){
				//fetch the last task of the connected opportunity
				$sql = "Select opportunity_id from documents_opportunities where document_id = '".$original_doc_bean->id."' and deleted = '0'";
				$result = $db->query($sql);
				$ans = $db->fetchbyassoc($result);
				var_dump($ans);
				if($ans){
					$opp_id = $ans['opportunity_id'];
					$opp_bean = BeanFactory::getBean('Opportunities',$opp_id);
					//var_dump($opp_bean->name);
					
					//var_dump($opp_bean->last_task_id_c);
					$task_bean = BeanFactory::getBean('Tasks',$opp_bean->last_task_id_c);
					//var_dump($task_bean->reupload_flag_c);
					if($task_bean->reupload_flag_c){
						//create the next task
						if($original_doc_bean->category_id == 'INITIAL_PROPOSAL')

						{		

		          				$task_id = $this->create_task($opp_bean->name." /Initial Proposal Approval",false,'INITIAL_PROPOSAL_APPROVAL',$opp_id,$opp_bean->assigned_user_id,$bean);

							$this->close_last_task($opp_bean->last_task_id_c,$now);

				                        $this->update_opportunity($opp_bean->id,'INITIAL_PROPOSAL_UPLOADED',$task_id,$_REQUEST['estimated_project_cost_c'],$_REQUEST['quotation_number_c']); 

			                        }
			                        
			                        if($original_doc_bean->category_id == 'DETAILED_DESIGN')

						{
						
							if($original_doc_bean->non_standard_item_c){

		          				$task_id = $this->create_task($opp_bean->name." /Detailed Design Feasibility Check",false,'DETAILED_DESIGN_FEASIBILITY',$opp_id,$opp_bean->user_id5_c,$bean);
		          				
		          				}else{
		          				$task_id = $this->create_task($opp_bean->name." /Customer Response On Detailed Design",false,'FINAL_PROPOSAL_RESPONSE',$opp_id,$opp_bean->user_id_c,$bean);
		          				
		          				}

				                        $this->close_last_task($opp_bean->last_task_id_c,$now);

							$this->update_opportunity($opp_bean->id,'',$task_id); 

			                        }
						
					
					}
				
				}
				
			
			}
			//fetch the opportunity for this document
			
		}
		
		//exit();

	}
	
	
	
	function create_task($task_name,$upload_flag,$task_flag,$opp_record_id,$assigned_user_id,$bean){

	

		$enddate = new DateTime($bean->date_modified, new DateTimeZone('Asia/Kolkata'));

		$enddate->add(new DateInterval("PT24H"));

		

		$new_task = BeanFactory::newBean('Tasks');

		$new_task->name = $task_name;

		$new_task->parent_type = 'Opportunities';

		$new_task->parent_id = $opp_record_id;

		$new_task->date_start = $bean->date_modified;

		$new_task->date_due = $enddate->format('Y-m-d H:i:s');

		$new_task->priority = 'High';

		$new_task->task_flag_c = $task_flag;

		$new_task->assigned_user_id = $assigned_user_id;

		if($upload_flag){

			$new_task->upload_task_c = '1';

		}

		$new_task->id = $new_task->save();

		return $new_task->id;

	}

	

	

	function close_last_task($opp_last_task_id,$now){

	

		$task_bean = BeanFactory::getBean('Tasks',$opp_last_task_id);

		if($task_bean){

		$task_bean->status='Completed';

		$task_bean->date_due=$now->format('Y-m-d h:i:s');

		$task_bean->save();

		}

	}

	

	

	function update_opportunity($opp_id, $sales_stage = '',$last_task_id,$estimation_cost='',$quotation_no=''){

	

		$opp_bean = BeanFactory::getBean('Opportunities',$opp_id);

		if($opp_bean){

			if($sales_stage != ''){

				$opp_bean->sales_stage = $sales_stage;

			}
			
			if($estimation_cost != '' && $quotation_no != ''){
			
				$opp_bean->quotation_number_c = $quotation_no;
				$opp_bean->estimated_project_cost_c = $estimation_cost;
			
			}

			$opp_bean->last_task_id_c = $last_task_id;

			$opp_bean->save();

		}

	}

	
	
}

?>
