Identification
==============

clc;
clear all;
close all;

%loading input and output data of the plant here
d1=load('');
%any dataset can be loaded here

%extracting the inputs and outputs from the loaded file
%select the appropriate columns from the loaded file to extract the input
%and output
u=d1(:,1);%input to the plant that is used for identification
y=d1(:,2);%output of the plant
u=u';
y=y';

%%Preprocessing operations here

%Filtering
%Do it for both input and output or one of them depending on the
%requirements
[num,den]=butter(2,0.69);
y=filter(num,den,y);

%Removing outliers
[y] = removeoutliers(y);
[u] = removeoutliers(u);

%depending on the data, appropriate preprocessing operation has to be
%performed.

Tsamp=0.0242;
T=(Tsamp*length(u))-(2*Tsamp);

%Identification begins here. 
m=2;%m and n are highest coefficients of s in numerator and denominator
n=2;
t1=0;
t2=T;
[a,b]=lse(u,y,Tsamp,t1,t2,m,n); %identifying the model
%Estimated parameters obtained from identification are a and b.

%This part is not essential but it helps to visualise the parameters in the form of a transfer function
%Creating a transfer function from the estimated parameters
num=b(1,1);
for i=2:m
    num=[num b(i,1)];
end
den=1;
for i=1:n
    den=[den a(i,1)];
end
sys_discrete=tf(num,den,Tsamp);
sys=zpk(sys_discrete);

%Subjecting the estimated model to an input and checking the response

ucheck=u; %estimated model is subjected to the input that the physical plant was subjected to
for i=1:n
    ini(i)=y(i);
end
y1est=estmodel(a,b,ucheck,Tsamp,T,ini);

%plotting the responses
t=0:Tsamp:T;
figure(1)
subplot(211)
plot(t,ucheck(1:length(t)))
xlabel('Time in seconds')
ylabel('Reactivity')
subplot(212)
plot(t,y(1:length(t)),'b',t,y1est(1:length(t)))
legend('Output of the physical plant','Output of the identified model')
xlabel('Time in seconds')
ylabel('Neutron Density')

