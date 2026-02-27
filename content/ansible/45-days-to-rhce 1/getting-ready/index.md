---
title: Getting Ready
date: 2026-02-01
series: ["45 Days to RHCE"]
series_order: 1
draft: false
---
![](featured.png)


This series is to document my final 47 days prepping for the RHCE exam. I have been using Ansible already for 11 months and have configured dozens of servers with Ansible. As a reference point. 

Make sure to sign up to my newsletter if you want updates on this series. I plan on releasing 1 article per week:
  <script async src="https://eomail5.com/form/d9b7d338-dbf3-11f0-bae4-65187d72ac9a.js" data-form="d9b7d338-dbf3-11f0-bae4-65187d72ac9a">
</script>

Let's make sure we have everything we need set up for getting RHCE certified.
## Scheduling the exam
First step I needed to take was [scheduling the exam](https://rhtapps.redhat.com/individualexamscheduler/#/Dashboard). I picked a date a couple weeks before my voucher expires. Just in case I fail, I want to be able to study hard for two weeks. Then do a retake. 

![](Pasted%20image%2020260201052356.png)

## Course materials
My only study material is this [book](https://www.amazon.com/RHCE-EX294-Cert-Guide-Certification/dp/0136872433/ref=sr_1_2?crid=1ZMN9NQWZA8HS&dib=eyJ2IjoiMSJ9.i-ddoTpDl9FtL3_grGGJeWUfBjh3voBjqLWIjzLKC0LQpybI3wGJH985YNmRx-v3Lr49AztXZAlNeFJ8HybfEpnbzviDpgZiz3iyNZC_u7d7Y9CTE6JreVpV-kZS6DeLB-ZpPe_M2ijMnczPiVNCPjo0hWJxe7whmw6GdgI-NK-s9s1s33HecRHixxsIxrDahiX5TLH5EgEADTbxVQ5vfD62nAlrJmLsNMrWPLrJgSA.KOdBHcqwRu1KsIRc3SOlI3bGR6eULH7QxJMns4BiTUQ&dib_tag=se&keywords=rhce+exam+guide&qid=1769951529&sprefix=rhce+exam+guid%2Caps%2C196&sr=8-2) and documentation that will be available during the exam. 
  <img src="rhceexamguide.png" class="grid-w20" />

## Study calendar
I like to use a spaced-repetition calendar to know what to study and when. When following this calendar, I'll be checking off each box as I do a review. 

Once I make it halfway down the first column, I will start down the second column. If an even number is reached, then I go to the next column. if an odd number is reached, then I go back to the previous column. 

This concept is hard to explain. But it'll look something like this:

|     | Topic                                                                                                              | Review 1 | Review 2 | Review 3 |
| --- | ------------------------------------------------------------------------------------------------------------------ | -------- | -------- | -------- |
| 1   | [Ad Hoc Commands](/notes/rhce-notes/adhoccommands/index.md)                                                        | X        | X        | X        |
| 2   | [ansible.cfg](../../../../notes/rhce-notes/ansibleconfig/index.md)                                                    | X        | X        | X        |
| 3   | [Ansible Vault](../../../../notes/rhce-notes/ansiblevault/index.md)                                                   | X        | X        | X        |
| 4   | [Boot Process](../../../../notes/rhce-notes/bootprocess/index.md)                                                     | X        | X        |          |
| 5   | [Deploying Files](../../../../notes/rhce-notes/deployingfiles/index.md)                                               | X        | X        |          |
| 6   | [Handlers](../../../../notes/rhce-notes/handlers/index.md)                                                            | X        | X        |          |
| 7   | [Hostname Patterns](../../../../notes/rhce-notes/hostnamepatterns/index.md)                                           | X        |          |          |
| 8   | [Including and Importing Files](../../../../notes/rhce-notes/includingandimportingfiles/index.md)                     | X        |          |          |
| 9   | [Inventory](../../../../notes/rhce-notes/inventory/index.md)                                                          | X        |          |          |
| 10  | [Jinja2 Templates](../../../../notes/rhce-notes/jinja2templates/index.md)                                             | X        |          |          |
| 11  | [Loops and Items](../../../../notes/rhce-notes/loopsanditems/index.md)                                                | X        |          |          |
| 12  | [Optimizing Ansible Processing](../../../../notes/rhce-notes/optimizingansibleprocessing/index.md)                    | X        |          |          |
| 13  | [Packages Repositories and Subscriptions](../../../../notes/rhce-notes/packagesrepositoriesandsubscriptions/index.md) |          |          |          |
| 14  | [Playbooks](../../../../notes/rhce-notes/playbooks/index.md)                                                          |          |          |          |
| 15  | [Roles](../../../../notes/rhce-notes/roles/index.md)                                                                  |          |          |          |
| 16  | [SeLinux](../../../../notes/rhce-notes/selinux/index.md)                                                              |          |          |          |
| 17  | [Services](../../../../notes/rhce-notes/services/index.md)                                                            |          |          |          |
| 18  | [Storage Partitions and LVM](../../../../notes/rhce-notes/storagepartitionsandlvm/index.md)                           |          |          |          |
| 19  | [Tags](../../../../notes/rhce-notes/tags/index.md)                                                                    |          |          |          |
| 20  | [Troubleshooting](../../../../notes/rhce-notes/troubleshooting/index.md)                                              |          |          |          |
| 21  | [Users and Groups](../../../../notes/rhce-notes/usersandgroups/index.md)                                              |          |          |          |
| 22  | [Variables](../../../../notes/rhce-notes/variables/index.md)                                                          |          |          |          |
| 23  | [When](../../../../notes/rhce-notes/when/index.md)                                                                    |          |          |          |
| 24  | [Practice Exam](../crash-and-burn/index.md)                                                                    |          |          |          |
## Time-blocking study sessions
This will be a new thing for me. But I am going to schedule my ideal daily study sessions and 1 longer breakout session per week. These sessions will be during the times where I am most likely to have the mental energy to study. 

Though I am not limited to only these times. And may study more based on energy levels and motivation.

![](Pasted%20image%2020260201060753.png)

## Understand the exam environment
You probably already know this since you are an RHCSA, but RedHat will email you instructions a day or two before the exam.  Check the PDF on this [page](https://learn.redhat.com/t5/Certification-Resources/Getting-Ready-for-your-Red-Hat-Certification-Remote-Exam/ba-p/33528) to get the latest exam requirements documentation. 

The ISO for the exam USB is also linked in that document. I'll be taking the exam remotely. 

## Study music
Study with me! Here is a public [RHCE Study Playlist](https://www.youtube.com/watch?v=lRezaTVzkGk&list=PLz70mC333biez9SVtFiqdhywt4tqVNMze) that you can add your favorite study tunes to. 

I am also considering live-streaming study sessions, [let me know](https://www.davidwrites.xyz/contact/) if I should!

## Practice exam
I’ll be working through this [practice exam](https://www.lisenet.com/2019/ansible-sample-exam-for-ex294/) and documenting where I find the answers.

## Building a lab
I'm using [LibVirt and ansible](https://www.davidwrites.xyz/articles/build-an-ansible-lab-with-ansible/) to build and Deploy virtual machines for my lab. Also, the Virt-manager app as a GUI for other VM management tasks. 

The git repo for my Ansible lab can be found [here.](https://github.com/davidwritesxyz/ansible)

## Finding answers during the exam

The devil is in the details. We need to make sure we know where to find answers on the fly during the exam. 

This would be a super easy exam if you had unlimited time. Because you have access to all the official documentation.

- [Ansible documentation site](https://docs.ansible.com/)
- [Ansible-doc command](https://docs.ansible.com/ansible/latest/cli/ansible-doc.html)
- Man pages

For practice exams or topics that we need more details, it's going to be super-important to use this documentation to find answers instead of course material or notes. 

This way, we'll get used to finding answers quickly, which we'll need to do on game day. 

That's it for now!



