# fortran-code
fortran 大作业
program expand_1D2D
    character*80 cmdfile,input_1Dfile,input_2Dfile,output_1Dfile,output_2Dfile
    integer input_type,expand_method
    real factor_m,factor_n,eigval
    cmdfile='expand.par'
    call read_par(cmdfile,input_1Dfile,input_2Dfile,output_1Dfile,output_2Dfile,eigval,factor_m,factor_n,input_type,expand_method)
   
   
    if(input_type==1)then
        write(*,*)'1D剖面数据扩边'
        call expand_1Ddat(input_1Dfile,expand_method,factor_m,eigval,output_1Dfile)
    elseif(input_type==2)then
        write(*,*)'2D剖面数据扩边'
        call expand_2Ddat(input_2Dfile,expand_method,factor_m,factor_n,eigval,output_2Dfile)
    endif
    
    end program
    
    !读入par文件
    subroutine read_par(cmdfile,input_1Dfile,input_2Dfile,output_1Dfile,output_2Dfile,eigval,factor_m,factor_n,input_type,expand_method)
        character*(*) cmdfile,input_1Dfile,input_2Dfile,output_1Dfile,output_2Dfile
        character*80 str
           integer input_type,expand_method
           real factor_m,factor_n,eigval
           open(10,file=cmdfile,status='old')
           read(10,*)str,eigval
           read(10,*)str,input_type
           read(10,*)str,expand_method
           read(10,*)str,factor_m
           read(10,*)str,factor_n
           read(10,*)str,input_1Dfile
           read(10,*)str,input_2Dfile
           read(10,*)str,output_1Dfile
           read(10,*)str,output_2Dfile
           write(*,*)eigval,factor_m,factor_n,input_1Dfile,input_2Dfile,output_1Dfile,output_2Dfile
           close(10)
    end subroutine
    !1D扩边
    subroutine expand_1Ddat(input_1Dfile,expand_method,factor_m,eigval,output_1Dfile)
        character*(*)input_1Dfile,output_1Dfile
         real factor_m,eigval,xmin,xmax,xxmin,xxmax,dx
         integer expand_method,point,m,m1,m2
         real,allocatable::expand1(:),x(:)
         call input_dat_para(input_1Dfile,point)!算出点数
         
        
         call cal_parameter1(point,m1,m2,m,factor_m)
         allocate(expand1(m),x(m))
         call input_dat(input_1Dfile,expand1,x,m,m1,m2,eigval,xmin,xmax,xxmin,xxmax,dx)! 将dat文件中的数据读入数组（专用于扩边前）
         call expand_1D(expand1,m,m1,m2,expand_method)!插值
         call output_dat(output_1Dfile,expand1,x,m,xxmin,dx)
         deallocate(expand1,x)
    end subroutine
    
    subroutine input_dat_para(input_1Dfile,point)
         character*(*)input_1Dfile
         integer point
         point=0
         open(60,file=input_1Dfile,status='old')
         do while(.not.eof(60))
             point=point+1
             read(60,*)
         enddo
         close(60)
         write(*,*)'point',point
    end subroutine
    
    
    
    subroutine cal_parameter1(point,m1,m2,m,factor_m)
         integer point
         real factor_m
         K_point=int(LOG(real(point))/LOG(2.0)+factor_m)
         m=2**(K_point)
         m1=(m-point)/2+1
         m2=m1+point-1
          write(*,*)'m=',m,'m1=',m1,'m2=',m2
    end subroutine
    
    subroutine  input_dat(input_1Dfile,expand1,x,m,m1,m2,eigval,xmin,xmax,xxmin,xxmax,dx)
         character*(*) input_1Dfile
         integer m,m1,m2
         real expand1(m),x(m)
         real eigval,xmin,xmax,xxmin,xxmax,dx
         do i=1,m
	         expand1(i)=eigval
         enddo
         open(60,file=input_1Dfile,status='old')
         do i=m1,m2
             read(60,*)x(i),expand1(i)
         enddo
         xmin=x(m1)
         xmax=x(m2)
         dx=(xmax-xmin)/(m2-m1)
         xxmin=xmin-(m1-1)*dx
         xxmax=xmax+(m-m2)*dx
         write(*,*)'dx=',dx,'xxmin=',xxmin,'xxmax=',xxmax
         close(60)
    end subroutine
    
    subroutine expand_1D(expand1,m,m1,m2,expand_method)
        integer m,m1,m2,expand_method
        real expand1(m)
        real sum,ave
        if(expand_method==1)then
            ave=0.0
        elseif(expand_method==2)then
            sum=0.0
            sum=sum+expand1(m1)+expand1(m2)
            ave=sum/((m2-m1+1)*2)
        elseif(expand_method==3)then
            sum=0.0
                do i=m1,m2
                    sum=sum+expand1(i)
                enddo
            ave=sum/(m2-m1+1)
        endif
       expand1(1)=ave
       expand1(m)=ave
       do i=2,m1-1
          expand1(i)=expand1(1)+(expand1(m1)+expand1(1))*cos((m1-i)/(m1-1)*3.1415926/2)
       enddo
        
       do i=m2+1,m-1
           expand1(i)=expand1(m2)+(expand1(m)+expand1(m2))*cos((m-i)/(m-1)*3.1415926/2)
       enddo
    endsubroutine
    
    subroutine output_dat(output_1Dfile,expand1,x,m,xxmin,dx)
         character*(*)output_1Dfile
        integer m
        real expand1(m),x(m)
        real xxmin,dx
        write(*,*)'xxmin',xxmin,'dx=',dx
        do i=1,m
            x(i)=xxmin+(i-1)*dx
        enddo
        do i=1,m
            write(*,*)x(i)
        enddo
        
        open(70,file=output_1Dfile,status='unknown')
        do i=1,m
            write(70,*)x(i),expand1(i)
        enddo

        close(70)
    end subroutine
    
 subroutine expand_2Ddat(input_2Dfile,expand_method,factor_m,factor_n,eigval,output_2Dfile)
         character*(*)input_2Dfile,output_2Dfile
         real factor_m,factor_n,eigval
         integer expand_method,point
         real,allocatable::expand2(:,:)
         call input_grd_para(input_2Dfile,point,line,Xmin,Xmax,Ymin,Ymax)!读入扩边前.grd
         call cal_parameter(point,line,Xmin,Xmax,Ymin,Ymax,m1,m2,n1,n2,m,n,dx,dy,factor_m,factor_n,XXmin,XXmax,YYmin,YYmax)
         allocate(expand2(m,n))
         call input_grd(input_2Dfile,expand2,m,n,m1,m2,n1,n2,eigval)! 将grd文件中的数据读入数组（专用于扩边前）
         call expand_2D(expand2,m,n,m1,m2,n1,n2,expand_method)!插值
         call output_grd(output_2Dfile,expand2,m,n,XXmin,XXmax,YYmin,YYmax)
         deallocate(expand2)
    end subroutine
    
    subroutine input_grd_para(input_2Dfile,point,line,Xmin,Xmax,Ymin,Ymax)
          character*(*)input_2Dfile
          integer point
          integer line
          real Xmin,Xmax,Ymin,Ymax
          open(20,file=input_2Dfile,status='old')
          read(20,*)
          read(20,*)point,line
          read(20,*)Xmin,Xmax
          read(20,*)Ymin,Ymax
          close(20)
    end subroutine
    
    subroutine cal_parameter(point,line,Xmin,Xmax,Ymin,Ymax,m1,m2,n1,n2,m,n,dx,dy,factor_m,factor_n,XXmin,XXmax,YYmin,YYmax)
          integer point,line,m1,m2,n1,n2,m,n
          real Xmin,Xmax,Ymin,Ymax,dx,dy,factor_m,factor_n,XXmin,XXmax,YYmin,YYmax
          K_point=int(LOG(real(point))/LOG(2.0)+real(factor_m))
	      K_line=int(LOG(real(line))/LOG(2.0)+real(factor_n))
      m=2**(K_point)       
      n=2**(K_line)
      m1=(m-point)/2+1
      m2=m1+point-1
      n1=(n-line)/2+1
      n2=n1+line-1
     dx=(Xmax-Xmin)/(point-1)
     dy=(Ymax-Ymin)/(line-1)

     XXmin=Xmin-(m1-1)*dx
     XXmax=Xmax+(m-m2)*dx
     YYmin=Ymin-(n1-1)*dy
     YYmax=Ymax+(n-n2)*dy
    END SUBROUTINE 
    
    subroutine input_grd(input_2Dfile,expand2,m,n,m1,m2,n1,n2,eigval)
        character*(*) input_2Dfile
        real expand2(m,n)

     do j=1,n
          do i=1,m
	         expand2(i,j)=eigval
         enddo
     enddo

      open(30,file=input_2Dfile,status='old')
      read(30,*)
      read(30,*)
      read(30,*)
      read(30,*)
      read(30,*)
      read(30,*)((expand2(i,j),i=m1,m2),j=n1,n2)
      close(30) 

    END SUBROUTINE
    
    subroutine expand_2D(expand2,m,n,m1,m2,n1,n2,expand_method)
        integer m,n,m1,m2,n1,n2,expand_method
        real expand2(m,n)
        real sum,ave
        !求平均值并给边界赋值
        if(expand_method==1)then
            ave=0.0
        elseif(expand_method==2)then
            sum=0.0
            do i=m1,m2
                sum=sum+expand2(i,n1)+expand2(i,n2)
            enddo
            do j=n1,n2
                sum=sum+expand2(m1,j)+expand2(m2,j)
            enddo
            ave=sum/(((n2-n1+1)+(m2-m1+1))*2)
        elseif(expand_method==3)then
            sum=0
            do j=n1,n2
                do i=m1,m2
                    sum=sum+expand2(i,j)
                enddo
            enddo
            ave=sum/((n2-n1+1)*(m2-m1+1))
        endif
        do i=1,m
            expand2(i,1)=ave
            expand2(i,n)=ave
        enddo
        
        do j=1,n
            expand2(1,j)=ave
            expand2(m,j)=ave
        enddo
        write(*,*)'ave',ave
        do j=n1,n2
            do i=2,m1-1
                expand2(i,j)=expand2(1,j)+(expand2(m1,j)-expand2(1,j))*cos((m1-i)/(m1-1)*3.1415926/2.0)
            enddo
        enddo
        do j=n1,n2
            do i=m2+1,m-1
                expand2(i,j)=expand2(m2,j)+(expand2(m,j)-expand2(m2,j))*cos((m-i)/(m-m2)*3.1415926/2.0)
            enddo
        enddo
        do i=1,m
            do j=2,n1-1
                expand2(i,j)=expand2(i,1)+(expand2(i,n1)-expand2(i,1))*cos((n1-j)/(n1-1)*3.1415926/2.0)
            enddo
        enddo
        do i=1,m
            do j=n2+1,n-1
                expand2(i,j)=expand2(i,n2)+(expand2(i,n)-expand2(i,n2))*cos((n-j)/(n-1)*3.1415926/2.0)
            enddo
        enddo
        
    end subroutine
    
    subroutine output_grd(output_2Dfile,expand2,m,n,XXmin,XXmax,YYmin,YYmax)
        character*(*) output_2Dfile
        
        integer m,n
        real expand2(m,n)
        real XXmin,XXmax,YYmin,YYmax
        real Gmin,Gmax
        open(50,file=output_2Dfile,status='unknown')
        Gmin=HUGE(Gmin)   !给Gmin赋予最大值
        Gmax=-HUGE(Gmax)  !给Gmax赋予最小值
        Do j=1,n
             Do i=1,m
                Gmin=MIN(Gmin,expand2(i,j))
                Gmax=MAX(Gmax,expand2(i,j))
             End do
        End do
write(50,'(a)')'DSAA'
write(50,*)m,n
write(50,*)XXmin,XXmax
write(50,*)YYmin,YYmax
write(50,*)Gmin,Gmax
do j=1,n
    write(50,*)(expand2(i,j),i=1,m)
end do
close(50)  	  	 
END SUBROUTINE
