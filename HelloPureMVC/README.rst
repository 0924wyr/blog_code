============================
����pureMVC�ļ�����������
============================


��Ϊ������Ҫ,���Ҳ��ʼѧϰ��ʹ�� `pureMVC`_ , ֮ǰҲд��һ�� `pureMVC Hello World�̳�`_,
���ǿ�ʼ���������ܺ�AS��������.�������ܶ��ѧϰ��ʹ��,�����˼������ʺ͹�˾����AS��ͬ��
��ͨʱ,������Ҳ�����������˵������Ȼ��,���Ծ��������о���Щ����,�����Ը������.

������������漸��:

1. ���һ��notification�ж��������,��ô����������֮ǰ��ִ��˳������ô����?
2. һ�δ�����ִ��ʱ,�������һ��notification,�ǵȴ���Ӧ��notification�����ߴ�������ټ�����,����
   ֱ�Ӽ���,��Ӧ��notification�������첽��ִ��?
3. һ��notification�Ķ��������֮���ִ�����첽��,����˳��ִ�е�?
4. �Ƿ���ܳ���notification��ѭ��������? (��notification A�Ķ�����X����notification B,B�Ķ�����
   �ַ���notification A,�γ�һ��û�г��ڵ�ѭ��)

.. image:: ../../images/notification.jpg

�о��ͽ����������
======================

Ϊ��׼ȷ�о�������������ʹ�,�Ҳ��õķ�����������: ʵ���Դ�����Ķ�(��ʵֻҪ�Ķ�Դ���뼴��).

ʵ��
----------

��صĴ���ɴ� `TestPureMVC`_ ����.

���������Ҫ�����µ�����:

Startup -> StartUpCommand -> registerMediator(AMediator, BMediator, CMediator) ->
sendNotification(Test) -> CMediator(Test) -> sendNotification(Second) ->CMediator(Second)
->sendNotification(Test)

�Ӷ��γ���һ��ѭ��,����,Ҳ�ɼ򵥵���StartUpCommand���и������������������.

�Ӵ������ܹ�˵��(��Ӧ�ڱ��������4������):

1. notification�Ķ�������ߵ�ִ��˳���ǰ�����ע���˳��ִ�е�,Ҳ����ȫ�ֵ�view��ά����mediator�����е�˳��
2. ��ΪAS��û��������sleep�ķ���,�����޷�ȷ��2(�������ο�����,�����������)
3. ͬ2
4. �������ѭ��,���ջ����ջ����Ĵ���

Դ�������
-----------------

��ΪCommand��Mediator�����Դ���ͷ���notification,��������ֻ��Mediator�Ĵ���Ϊ����˵��.

�����ȿ�notification�����֪ͨ��:

::

    // org.puremvc.as3.core.Views.as
    public function notifyObservers( notification:INotification ) : void
    {
        if( observerMap[ notification.getName() ] != null ) {
            
            // Get a reference to the observers list for this notification name
            // ������notification�����ж���������
            var observers_ref:Array = observerMap[ notification.getName() ] as Array;

            // Copy observers from reference array to working array, 
            // since the reference array may change during the notification loop
            //  notification��ѭ���п��������µĶ�����,����������ȿ���һ��
            // ע��˳��û�и���
            var observers:Array = new Array(); 
            var observer:IObserver;
            for (var i:Number = 0; i < observers_ref.length; i++) { 
                observer = observers_ref[ i ] as IObserver;
                observers.push( observer );
            }
            
            // Notify Observers from the working array				
            // ���������е�˳��������֪ͨ��Ӧ�Ķ�����
            // ע��������˳��ִ�е�
            for (i = 0; i < observers.length; i++) {
                observer = observers[ i ] as IObserver;
                observer.notifyObserver( notification );
            }
        }
    }

���������¼������ע���,�Ӷ��γɶ����ߵ�����:

::

    // org.puremvc.as3.core.Views.as
    public function registerMediator( mediator:IMediator ) : void
    {
        // do not allow re-registration (you must to removeMediator fist)
        if ( mediatorMap[ mediator.getMediatorName() ] != null ) return;
        
        // Register the Mediator for retrieval by name
        mediatorMap[ mediator.getMediatorName() ] = mediator;
        
        // Get Notification interests, if any.
        var interests:Array = mediator.listNotificationInterests();

        // Register Mediator as an observer for each of its notification interests
        if ( interests.length > 0 ) 
        {
            // Create Observer referencing this mediator's handlNotification method
            var observer:Observer = new Observer( mediator.handleNotification, mediator );

            // Register Mediator as Observer for its list of Notification interests
            for ( var i:Number=0;  i<interests.length; i++ ) {
                // ע��: �����Ǹ���Mediator����Ȥ��notification���ֱ���뵽��Ӧ�Ķ�����������
                // ������Ĵ��������Ǹ������Mediator��ע��˳��
                registerObserver( interests[i],  observer );
            }			
        }
        
        // alert the mediator that it has been registered
        mediator.onRegister();
        
    }

    // registerObserver�ľ���ʵ��
    public function registerObserver ( notificationName:String, observer:IObserver ) : void
    {
        var observers:Array = observerMap[ notificationName ];
        if( observers ) {
            // ˳����뵽��Ӧ�Ķ�����������
            observers.push( observer );
        } else {
            observerMap[ notificationName ] = [ observer ];	
        }
    }


    

�����ٿ�֪ͨ��������ʱ�Ĵ����߼�:

::

    public function notifyObserver( notification:INotification ):void
    {
        // ����notification����������������ִ����Ӧ�Ĵ����߼�
        this.getNotifyMethod().apply(this.getNotifyContext(),[notification]);
    }
	
����,�����Ƕ�Դ����ķ�����,���ǾͿ��������Ļش��ĳ�ʼ��4������:

1. ���һ��notification�ж��������,��ô���������֮���ǰ���ע���˳����ִ�е�
2. ��AS�в������첽��ִ��,����,��ǰ�Ĵ����ִ�л�ȴ����е�notification����1��
   ��˳��ִ����ɺ�,�ſ�ʼ����ִ�е�ǰ�Ĵ���(�൱�ڵ���һ������)
3. һ��notification�Ķ��������֮����˳��ִ�е�,˳���ǰ���1�е�˵��
4. ������ѭ���Ŀ���,��Ϊ��2��˵��,sendNotification�൱�ڸ���ע���˳����˳��ִ��
   ��Ӧ�Ĵ����߼�,����ڴ����߼����ְ�������sendNotification���¼�,������ִ��
   ���Ϊһ����ѭ��,�Ӷ�����ջ���
    

�ܽ�
===========

���,ͨ���Ա��Ŀ�ʼ4������ķ���,Ū����� `pureMVC`_ ���ĵ�notification���Ƶļ���
��������,���ں����Ĺ�����ѧϰ���Ǻ����洦��.



