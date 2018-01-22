# CustomizedEmailAlertsTFSPlugin
/* In this post I will show you how to send Customized Email Alerts using TFS API. This will be done by creating a subscription to #WorkItemChanged event. Our code uses class represents an event handler that is subscribed to TFS event notifications.

Visual Studio Project:
Visual Studio > new project > C# Class Library > Add reference Assemblies (TFS and CRM related)

Plugin deployment location:
ProgramFiles/\Microsoft Team Foundation Server 12.0\Application Tier\Web Services\bin\Plugins

Assemblies can be found in 
C:\Program Files (x86)\Microsoft Visual Studio 10.0\Common7\IDE\ReferenceAssemblies\v2.0\
And
C:\Program Files (x86)\Microsoft Visual Studio 10.0\Common7\IDE\ReferenceAssemblies\v4.0\
1.	using Microsoft.TeamFoundation.Client; 
2.	using Microsoft.TeamFoundation.Common; 
3.	using Microsoft.TeamFoundation.Framework.Server; */


       using System.Net.Mail;

       using Microsoft.TeamFoundation.Client;

       using Microsoft.TeamFoundation.Common;

       using Microsoft.TeamFoundation.Framework.Server;

       using System.IO;

       using System.Web;

       namespace CustomizedEmailNotificationPlugin

       {

/// This class represents an event handler that is subscribed to TFS event notifications

       public class Class1 : ISubscriber

       {

/// Specifies the TFS events that objects created from this class are subscribed to.

///<return>Set of types this subscriber wants to be notified of </return>

       public Type[] SubscribedTypes()

       {   return new Type[1] { typeof(WorkItemChangedEvent) };  }

       String State = String.Empty;

       String CreatedDate = String.Empty;

       String IssueSummary = String.Empty;

/// TFS Main Event Handler

/// <parameter name="requestContext">Event context passed in by TFS</parameter>

/// <parameter name="notificationType">DecisionPoint or Notification</parameter>

/// <parameter name="notificationEventArgs">Object that was published</parameter>

/// <parameter name="statusCode">Code to return to the user when a decision point fails</parameter>

/// <parameter name="statusMessage">Message to return to the user when a decision point fails</parameter>

/// <parameter name="properties">Properties to return to the user when a decision point fails</parameter>

/// <returns></returns>

        public EventNotificationStatus ProcessEvent(TeamFoundationRequestContext requestContext, NotificationType notificationType, object 

        notificationEventArgs,

        out int statusCode, out string statusMessage, out ExceptionPropertyCollection properties)

        {

        statusCode = 0;

        properties = null;

        statusMessage = String.Empty;

        try

        {

        if (notificationType == NotificationType.Notification && notificationEventArgs is WorkItemChangedEvent)

        {

        WorkItemChangedEvent ev = notificationEventArgs as WorkItemChangedEvent;

        Uri tfsUri = new Uri("TYPE URL HERE");

        TfsConfigurationServer configurationServer =

        TfsConfigurationServerFactory.GetConfigurationServer(tfsUri);

        TfsTeamProjectCollection tpc = new TfsTeamProjectCollection(tfsUri, new System.Net.NetworkCredential("TYPE USERNAME HERE", "TYPE 

        PASSWORD HERE", "TYPE DOMAIN HERE"));

        try

        {

        tpc.EnsureAuthenticated();

        }

        catch (Exception ex)

        {

        throw new Exception(string.Format("Auth fail.({0})", ex.Message));

        }

//From event context extract TFS ID of the work item and work item instance

       WorkItemStore workItemStore = tpc.GetService<WorkItemStore>();

       Project teamProject = workItemStore.Projects["TYPE TEAMPROJECT NAME HERE"];

       WorkItemType workItemType = teamProject.WorkItemTypes["TYPE WORKITEM NAME HERE"];

       int workItemId = ev.CoreFields.IntegerFields[0].NewValue;

       WorkItem IMRWorkItem = workItemStore.GetWorkItem(workItemId);



       String EmailID = String.Empty;

       String EmailIDCC = String.Empty;

       EmailID = "abc@account.com";

       EmailIDCC = "abc@account.com";



//Extracting Reference Field data of work item and saving it to variable for supplying it to email body

     CreatedDate = IMRWorkItem.Fields["System.CreatedDate"].Value.ToString();

     IssueSummary = IMRWorkItem.Fields["Microsoft.VSTS.CMMI.CMMI_Inc_IssueSummary"].Value.ToString();

     State = IMRWorkItem.Fields["System.State"].Value.ToString();


     if (State == "Active")

     {



       CreateEmail(EmailID, "Body","Subject", EmailIDCC);
       // We will use Customized Email procedure

       //Calling Email procedure
       }

       }

       }

       catch (Exception ex)

       {

       EventLog.WriteEntry("Exception in CustomizedEmailNotificationPlugin", ex.ToString());

       }

       return EventNotificationStatus.ActionPermitted;

       }

//Definition of Customized Email procedure.

//The MailMessage class represents the content of a mail message. The SmtpClient class transmits email to the SMTP host that you 

//designate for mail delivery. You can create mail attachments using the Attachment class.


          public void CreateEmail(string Toemail1, string sBody, string subject, string cc)

        {

        System.Net.Mail.SmtpClient mySmtpClientCureMDSupport = new System.Net.Mail.SmtpClient();

        mySmtpClientCureMDSupport.UseDefaultCredentials = true;

        mySmtpClientCureMDSupport.Port = 25;

        MailMessage message1 = new MailMessage();

        MailAddress fromAddress1 = new MailAddress("tfs.alerts@curemd.com");

        mySmtpClientCureMDSupport.Host = "enter smtp server name";

        mySmtpClientCureMDSupport.Credentials = new System.Net.NetworkCredential("enter smtp user name", "enter smtp password");

        message1.From = fromAddress1;

        message1.Subject = subject;

        if (Toemail1 != "")

        message1.To.Add(Toemail1);

        if (cc != "")

        {

        message1.CC.Add(cc);

        }
        

       message1.Body = CreateBody();

       message1.IsBodyHtml = true;

       mySmtpClientCureMDSupport.Send(message1);

       }
//message1.Body = sBody;

//We will use Customized email body

//Calling Customized email body procedure

//Definition of Customized email body procedure

       private string CreateBody()

       {

       string body = string.Empty;

       string filePath = "IMR.html";

       using (StreamReader reader = new StreamReader(filePath))

       {

       body = reader.ReadToEnd();

       }

       body = body.Replace("{createDate}", CreatedDate); //replacing parameters used in HTML file

       body = body.Replace("{IssueSummary}", IssueSummary);



       return body;

       }

/// <summary>

/// Friendly name of the Subscriber used in messages about the subscriber

/// </summary>

      public string Name

      {

      get { return "WorkItemChangedEventHandler"; }

      }

/// <summary>

/// Priority this subscriber is called on

/// </summary>

        public SubscriberPriority Priority

        {

        get { return SubscriberPriority.Normal; }

        }

        }

        }


